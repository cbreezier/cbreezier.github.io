---
title: Handling optional JSON fields for PATCH APIs with Jackson/Kotlin
tags:
  - programming
  - kotlin
date: 2025-06-08 22:36:31
---


When HTTP/1.0 was first proposed in 1996, it defined only 3 methods in [RFC 
1945](https://datatracker.ietf.org/doc/html/rfc1945#section-5.1.1): `GET`, 
`HEAD`, and `POST`.

Then [RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616#section-5.1.1) 
introduces many other methods including `PUT` and `DELETE` in 1999.

But it's not until 2010 that
[RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789) finally introduces 
the `PATCH` method.

Perhaps that's why it's often forgotten and rarely discussed.

## The `PATCH` method
> The existing
HTTP PUT method only allows a complete replacement of a document.
This proposal adds a new HTTP method, PATCH, to modify an existing
HTTP resource.
> 
> _[RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789)_

This method is really quite useful, and I would go so far as to say that the 
majority of REST-y CRUD-y "update" operations should use `PATCH` rather than 
`PUT`. It allows us to:
 - save on bandwidth by only submitting fields that change
 - minimise race conditions where the last updater wins (imagine a resource 
   with 1000 fields, but you only want to change 1 of them - using `PUT` you 
   essentially "lock" out and introduce race conditions on all 1000 fields)

There are multiple ways to implement a `PATCH` API - indeed, RFC 5789 
doesn't really specify an implementation.

In the JSON world, there are two main ways:
 - JSON Merge Patch ([RFC 7396](https://datatracker.ietf.org/doc/html/rfc7396))
 - JSON Patch ([RFC 6902](https://datatracker.ietf.org/doc/html/rfc6902))

If we have a resource that looks like this:

```json
{
  "name": "John Doe",
  "phone": "+61444555666"
}
```

and we want to modify just the name to "Johnny Doe", then a `PUT` API might 
look like:

```json
{
  "name": "Johnny Doe",
  "phone": "+61444555666"
}
```

a JSON Merge Patch would look like:

```json
{
  "name": "Johnny Doe"
}
```

and a JSON Patch would look like:

```json
[{
  "op": "replace",
  "path": "/name",
  "value": "Johnny Doe"
}]
```

We're going to focus on JSON Merge Patch, since it's the simplest, and 
sufficient in most cases.

## `undefined`: the black sheep
People love to hate on `undefined` vs `null` in JavaScript. Honestly though, 
I've never had an issue with it. Even though they may be often 
interchangeable, they clearly represent two semantically different concepts.

And nowhere is it clearer than with a JSON Merge Patch request.

Everything is simpler when all the fields in your resource are non-nullable, 
but when `null` is a valid value for your field, you need to clearly 
disambiguate between 3 states:
1. I want to update the `"name"` field (use a `string`)
2. I want to update the `"name"` field to null (use `null`)
3. I don't want to update the `"name"` field (omit the field entirely, ie 
   `undefined` semantics)

## Implementing `PATCH` APIs with Jackson / Kotlin
Unfortunately, it's not exactly easy to properly disambiguate between 
"missing field" and "null field" when deserializing with Jackson (or perhaps 
with any popular library - it certainly seems difficult with `kotlinx.
serialization` too).

### Level 1: `Optional<T>?`
There's actually a workable solution out of the box: a nullable optional field.

```kotlin
data class UserPatch(
  val name: Optional<String>?,
  val phone: String?,
)
```

Note that `java.util.Optional` does not allow containing `null` values, so 
`Optional<String?>` isn't an option.

Out of the box, this nullable optional field does kind of work:

|            | Value                | Null               | Undefined        |
|------------|----------------------|--------------------|------------------|
| Json       | `"name": "value"`    | `"name": null`     |                  |
| Kotlin     | `Optional.of(value)` | `Optional.empty()` | `null`           |
| Serialized | `"name": "value"`    | `"name": null`     | `"name": null` ❌ |

When deserializing (the important part as a webserver), we are correctly 
able to disambiguate between the three states. However:
 - it is _really_ confusing that `null` means `undefined`
 - when serializing, we still get `null` written out for the `undefined` 
   case, unless we ass an annotation like `@JsonInclude(Include.NON_NULL)` 
   on the field

### Level 2: custom `OptionalField<T?>`
We can define our own class that _does_ allow containing `null` values, and 
this gives it all the right semantics.

Here's what my implementation looks like:
```kotlin
/**
 * A class specifically intended to be used as a field in a JSON-serializable object.
 *
 * It can represent 3 cases:
 *  - a present value
 *  - a null value
 *  - an undefined value (field absent from the json entirely)
 */
sealed class OptionalField<out T> {

    // Such a nice API. Why doens't java.util.Optional implement this?
    abstract fun <R> fold(ifPresent: (T) -> R, ifUndefined: () -> R): R

    data class Present<T>(
        // This means that this field should be used directly for ser/deser,
        // rather than serializing this class as an object containing the value
        @JsonValue
        val value: T,
    ): OptionalField<T>() {
        override fun <R> fold(ifPresent: (T) -> R, ifUndefined: () -> R): R {
            return ifPresent(value)
        }
    }

    data object Undefined: OptionalField<Nothing>() {
        override fun <R> fold(ifPresent: (Nothing) -> R, ifUndefined: () -> R): R {
            return ifUndefined()
        }
    }
}

```

And this is what it looks like to use:

|            | Value                          | Null           | Undefined                 |
|------------|--------------------------------|----------------|---------------------------|
| Json       | `"name": "value"`              | `"name": null` |                           |
| Kotlin     | `OptionalField.Present(value)` | `null`         | `OptionalField.Undefined` |
| Serialized | `"name": "value"`              | `"name": null` |                           |

**Perfect**.

Except, of course, it's not that easy. Usually, writing custom serializers 
and deserializers for Jackson is pretty painless. However, when you want to 
change the behaviour of a serializer to _omit a field entirely_, you end up 
having to dive pretty deep into Jackson internals.

With a normal custom serializer, Jackson doesn't invoke your serializer 
until it's already written `"name":`.

So we need to jump in earlier, using a different technique. I won't bore 
you with the 4+ hour journey of cross-referencing ChatGPT hallucinations 
against documentation and ample testing, and just show you the final code.

Everything required for **serialization**:

```kotlin
import com.fasterxml.jackson.core.JsonGenerator
import com.fasterxml.jackson.databind.BeanDescription
import com.fasterxml.jackson.databind.SerializationConfig
import com.fasterxml.jackson.databind.SerializerProvider
import com.fasterxml.jackson.databind.ser.BeanPropertyWriter
import com.fasterxml.jackson.databind.ser.BeanSerializerModifier

/**
 * This is the entirety of the magic for serializing our optional field.
 *
 * We need to modify the serialization _before_ we even get to the value if we want to omit
 * the entire field (including the key name).
 *
 * That's what we're doing here - replacing the default property writer with one that knows to
 * omit the entire field if we have an undefined value.
 *
 * Note that we don't actually need a custom serializer for [OptionalField] 
 * itself.
 */
object OptionalFieldBeanSerializerModifier: BeanSerializerModifier() {
    override fun changeProperties(
        config: SerializationConfig,
        beanDesc: BeanDescription,
        beanProperties: MutableList<BeanPropertyWriter>,
    ): MutableList<BeanPropertyWriter> {
        return beanProperties.map {
            val type = it.type
            // This check may not even be strictly necessary - our custom property writer only
            // affects our special undefined value anyway
            if (OptionalField::class.java.isAssignableFrom(type.rawClass)) {
                OptionalFieldWriter(it)
            } else {
                it
            }
        }.toMutableList()
    }
}

class OptionalFieldWriter(
    delegate: BeanPropertyWriter,
): BeanPropertyWriter(delegate) {
    override fun serializeAsField(bean: Any?, gen: JsonGenerator?, prov: SerializerProvider?) {
        val value = this.get(bean)
        if (value is OptionalField.Undefined) {
            // Omit serializing this field
            return
        }
        super.serializeAsField(bean, gen, prov)
    }
}
```

Everything required for **deserialization**:

```kotlin
import com.fasterxml.jackson.databind.*
import com.fasterxml.jackson.databind.deser.Deserializers
import com.fasterxml.jackson.databind.deser.std.ReferenceTypeDeserializer
import com.fasterxml.jackson.databind.jsontype.TypeDeserializer
import com.fasterxml.jackson.databind.type.*
import java.lang.reflect.Type

/**
 * This is the bulk of the deserialization magic.
 *
 * A [ReferenceType] is a type that is a reference to a single other value (like our [OptionalField] type). Other special
 * types include [CollectionType] and [MapType].
 *
 * There's some very convenient methods here such as [getAbsentValue] that allow us to control how we deal with missing
 * fields vs null fields.
 */
class OptionalFieldDeserializer<T>(
    fullType: JavaType,
    valueDeserializer: JsonDeserializer<*>?,
    typeDeserializer: TypeDeserializer?,
): ReferenceTypeDeserializer<OptionalField<T>>(fullType, null, typeDeserializer, valueDeserializer) {

    override fun getNullValue(ctxt: DeserializationContext): OptionalField<T> {
        return OptionalField.Present(null as T)
    }

    override fun getAbsentValue(ctxt: DeserializationContext): OptionalField<T> {
        return OptionalField.Undefined
    }

    override fun referenceValue(contents: Any): OptionalField<T> {
        return OptionalField.Present(contents as T)
    }

    override fun getReferenced(reference: OptionalField<T>): T? {
        return reference.fold(
            { it },
            { null }
        )
    }

    override fun updateReference(reference: OptionalField<T>, contents: Any): OptionalField<T> {
        return OptionalField.Present(contents as T)
    }

    /**
     * Allows us to modify our deserializer for a different referenced type.
     */
    override fun withResolved(
        typeDeser: TypeDeserializer?,
        valueDeser: JsonDeserializer<*>?,
    ): ReferenceTypeDeserializer<OptionalField<T>> {
        return OptionalFieldDeserializer<T>(
            _fullType,
            valueDeser,
            typeDeser,
        )
    }
}

/**
 * Because our [OptionalField] type is generic, we need to construct our deserializer in context of
 * the parameterized type that we're actually deserializing. That's what this does - helps us find the right
 * deserializer for the right container ([OptionalField]) with the right content type defined.
 */
object OptionalFieldDeserializers: Deserializers.Base() {
    override fun findReferenceDeserializer(
        refType: ReferenceType,
        config: DeserializationConfig,
        beanDesc: BeanDescription,
        contentTypeDeserializer: TypeDeserializer?,
        contentDeserializer: JsonDeserializer<*>?,
    ): JsonDeserializer<*>? {
        return if (OptionalField::class.java.isAssignableFrom(refType.rawClass)) {
            OptionalFieldDeserializer<Any>(refType, contentDeserializer, contentTypeDeserializer)
        } else {
            null
        }
    }
}

/**
 * We need to tell Jackson that our [OptionalField] type is a [ReferenceType], or it won't even try and
 * invoke our [OptionalFieldDeserializers.findReferenceDeserializer].
 *
 * This type modifier promotes any simple [OptionalField] types into a known [ReferenceType].
 */
object OptionalFieldTypeModifier: TypeModifier() {
    override fun modifyType(
        type: JavaType,
        jdkType: Type,
        context: TypeBindings,
        typeFactory: TypeFactory,
    ): JavaType {
        // Make sure we don't try and promote again after it's already known to be a ReferenceType
        return if (type.rawClass == OptionalField::class.java && type is SimpleType && type.containedTypeCount() == 1) {
            ReferenceType.upgradeFrom(type, type.containedType(0))
        } else {
            type
        }
    }
}
```

Finally, how to hook it all up as a Jackson module:
```kotlin
import com.fasterxml.jackson.databind.module.SimpleModule

object OptionalFieldModule: SimpleModule() {
    override fun setupModule(context: SetupContext) {
        super.setupModule(context)
        context.addTypeModifier(OptionalFieldTypeModifier)
        context.addDeserializers(OptionalFieldDeserializers)
        context.addBeanSerializerModifier(OptionalFieldBeanSerializerModifier)
    }
}

// Use it like this, for example:
fun customObjectMapper(): ObjectMapper {
    return jacksonObjectMapper().apply {
        registerModule(Jdk8Module())
        registerModule(JavaTimeModule())
        
        // This is our bit
        registerModule(OptionalFieldModule)

        disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)

        disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
        disable(SerializationFeature.WRITE_DURATIONS_AS_TIMESTAMPS)
    }
}
```

### Level 3: Use `jackson-databind-nullable`
Yeah, after all that, I found that someone has done this already (of course).

But it was pretty hard to find:
[jackson-databind-nullable](https://github.com/OpenAPITools/jackson-databind-nullable)

Some things I like more about my version:
 - I wrote every line of it, so I know exactly what's going on (I just 
   _like_ this - a personal failing)
 - I think `OptionalField` makes a lot more sense than `JsonNullable` as a 
   class name
 - Mine has only the bare minimum. Take away anything and it stops working

But `jackson-databind-nullable`:
 - is used by a lot more people and is more battle-tested
 - is more than bare minimum, which makes me worry that I'm missing something