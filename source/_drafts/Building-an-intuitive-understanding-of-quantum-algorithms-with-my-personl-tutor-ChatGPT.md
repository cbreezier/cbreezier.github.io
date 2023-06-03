---
title: >-
  Building an intuitive understanding of quantum algorithms with my personl
  tutor: ChatGPT
tags:
---
I've always been fascinated by anything related to quantum theory; the results described by quantum mechanics seem to defy all rules of logic and rationality. Quantum computing is no exception. Even the most basic quantum algorithms promise performance that just doesn't seem possible.

I'm a classical software engineer by trade. My knowledge of quantumm theory is surface-level at best, and I've more or less given up on truly understanding quantum mechanics at a deep level. There is simply too much math (although an _actual_ computer scientist or mathematician would probably chuckle and shake their head at that) and prerequisite knowledge that I've failed to slog through time and time again.

However, I'm still determined to build some level of intuitive understanding of how quantum computing works, what it can actually do, and what its limitations are. In short, I need to reconcile this seemingly law-breaking phenomenon with my reality, before I go and off myself like those scientists from _The Three Body Problem_ (oops, minor spoiler?)

## Why ChatGPT?
As I mentioned, I've tried learning about quantum computing a couple of times now. Each time I've found good resources and made some progress in my understanding, but each time I've also hit the same roadblock: I simply didn't have the prerequisite knowledge in some area or another, which prevented me from continuing.

It was like trying to understand an essay in a different language, but I had to look up every 10th word in the dictionary. And then I had to look up half of the words in the definition for that original word. Except it might be even harder than that - I might not even know exactly what I needed to look up. It could be because the thing I don't understand is:
 - a mathematical symbol, which is hard to look up, especially because it might have different meanings in different mathematical conexts
 - a term (eg, "amplitude") which again has a specific meaning in quantum theory but a different one elsewhere
 - a concept that isn't usually defined on its own, but rather taught as part of a wider set of knowledge (eg, having to re-learn large portions of "linear algebra")
 - a very specific question that someone else is unlikely to have already asked, and would therefore yield no results in an internet search

I often wished that I had access to an expert in the field who could answer my specific questions. Someone who could provide a _personalised_ learning experience tailored just for me. Someone I could talk to and ask questions about things that I should proabbly already know without feeling embarrassed. Someone like...ChatGPT?

## Grover's algorithm
One of the common introductory algorithms in quantum computing is Grover's algorithm. I'll let ChatGPT explain what it is:

**_I want to understand Grover's algorithm. Can you give me an overview of what it is?_**
> Sure, I'd be happy to help you understand Grover's algorithm!
> 
> Grover's algorithm is a quantum algorithm that can be used to search an unsorted database with N elements in O(√N) time. This is exponentially faster than the classical counterpart, which requires O(N) time to perform the same search. Grover's algorithm was discovered by Lov Grover in 1996 and has become an important algorithm in the field of quantum computing.

Searching an unsorted database with `N` elements in `O(√N)` time? Something that should be literally impossible? Even the most basic logic easily proves that it's impossible: in order to be sure that an element isn't in the database, you must at _least_ look at each element once. How can you be sure it isn't the 166th element if you didn't even look at it?

Incidentally, the fact about how it was "discovered by Lov Grover in 1996" is the kind of thing that I _don't_ really trust ChatGPT with. It's very good at making up facts like that which sound correct but are totally wrong. In this case it was correct though.

I bite off my skepticism. ChatGPT continues:

> The algorithm works by creating a quantum superposition of all possible states in the database. It then uses the amplitude amplification technique to increase the probability of measuring the correct state. This technique involves applying a series of operations to the quantum state, which reflects it around the mean amplitude of the desired state. By repeating this process several times, the probability of measuring the correct state approaches unity, and the correct state can be identified with high confidence.
> 
> The efficiency of Grover's algorithm comes from the fact that the quantum superposition allows for parallel computation, and the amplitude amplification technique increases the probability of measuring the correct state. This allows the algorithm to search through the database much faster than classical algorithms.

Okay, so that was a lot of stuff. I vaguely understood about 20% of that, and I wanted to double check my understanding and revise some concepts.

**_Can you elaborate on what it means to create a superposition of all possible states in the database?_**
> Sure! In Grover's algorithm, creating a superposition of all possible states in the database means creating a quantum state that represents all possible states of the database at the same time. This is done using a quantum circuit that involves applying a series of Hadamard gates to each qubit in the input register.
> 
> For example, let's say we have a database of N=8 elements, and we represent each element as a binary string of length 3. We can create a quantum superposition of all possible states of the database by initializing the input register to the state |000⟩ and applying a Hadamard gate to each qubit:
> 
> ```
|000⟩ --> H|0⟩ ⊗ H|0⟩ ⊗ H|0⟩
        = (|0⟩ + |1⟩) ⊗ (|0⟩ + |1⟩) ⊗ (|0⟩ + |1⟩)
        = |000⟩ + |001⟩ + |010⟩ + |011⟩ + |100⟩ + |101⟩ + |110⟩ + |111⟩
```
> This quantum state represents a superposition of all possible states of the database, with each state having the same amplitude of 1/√8. The state can be thought of as a weighted sum of all possible states of the database, with each state contributing equally to the superposition.

I have to say, the example was extremely helpful for me to understand what it was talking about. It helped me remember that a superposition was simply a linear combination of "basis vectors", and that the "amplitude" was simply the probability that the entire superposition would collapse to that particular "basis vector" when measured.

I didn't really understand what was going on with the math other than the last line, and I never really understood what a Hadamard gate did. For now, I'm not too interested in that. I'm sure ChatGPT could explain more if I pressed it on that.

**_Can you elaborate on the amplitude amplification technique?_**
> Sure! Amplitude amplification is a technique used in Grover's algorithm to increase the amplitude of the desired state in a superposition of quantum states. The technique involves using a reflection operation to increase the amplitude of the desired state while decreasing the amplitudes of the other states.
> 
> The reflection operation used in amplitude amplification is based on two steps: inversion about the mean and phase flipping. The inversion about the mean step involves applying a unitary operation that reflects the quantum state around the mean amplitude of the desired state. This is followed by a phase flipping step that flips the phase of the desired state.
> 
> To understand how amplitude amplification works, let's consider a simple example where we want to search for a single element in a database of N=4 elements. We can create a quantum superposition of all possible states of the database using the Hadamard transform, which gives us:
> 
> ```
|s⟩ = (|00⟩ + |01⟩ + |10⟩ + |11⟩) / 2
```
> The desired state is represented by a single element, say |10⟩, and we want to increase its amplitude while decreasing the amplitudes of the other states. To do this, we apply a series of iterations of the amplitude amplification operation, which consists of the following steps:
> 
> 1. Apply the oracle, which flips the phase of the desired state:
> ```
|s⟩ --> -|s⟩ (if the desired state is present in the database)
```
> 2. Apply the inversion about the mean, which reflects the quantum state around the mean amplitude of the desired state:
> ```
|s⟩ --> 2|10⟩ - |s⟩
```
> 3. Repeat steps 1 and 2 for a certain number of iterations, which amplifies the amplitude of the desired state while decreasing the amplitudes of the other states.
>
> By repeating this process for O(√N) iterations, we can increase the probability of measuring the desired state to near unity. This makes Grover's algorithm exponentially faster than classical algorithms for searching unsorted databases.

Alright, what the _flip_ is going on here? I have no idea what it's talking about with respect to the "oracle", "phase" or "phase flipping", or "inversion about the mean".

In particular, this "oracle" thing sounds super fishy. The whole point is to find a particular element in an unsorted database. And yet, one of the steps is to just magically "flip the phase" of the actual element that we're trying to find? That kind of sounds like "to bake a cake, first buy a cake from the supermarket".

I decide to assume that this algorithm is actually useful and not just an elaborate joke, and leave my question about the oracle til later.

**_Can you elaborate on "Apply the oracle, which flips the phase of the desired state"?_**
> Certainly! The oracle in Grover's algorithm is a unitary operator that performs a phase flip on the desired state in the superposition. The phase flip involves multiplying the amplitude of the desired state by -1, which changes the sign of the phase of the state. This has the effect of "marking" the desired state by making it distinguishable from the other states in the superposition.
> 
> In the context of the example I gave earlier, where we are searching for a single element |10⟩ in a database of N=4 elements, the oracle would perform the following transformation on the superposition:
> 
> ```
|10⟩ --> -|10⟩
```
> and leave all the other states unchanged. This is equivalent to flipping the phase of the amplitude of the state |10⟩, which marks it as the desired state.
> 
> The oracle can be implemented using various techniques, depending on the structure of the problem being solved. In general, the oracle is designed to take advantage of the structure of the problem to mark the desired state in the superposition. The choice of oracle is crucial to the success of Grover's algorithm, since it determines how the algorithm can "see" the desired state in the superposition and exploit it to increase its probability of being measured.

This was quite illuminating. I'm sure that there's some gross oversimplifications here, but it certainly sounds like "flipping the phase" is simply making the amplitude negative. I expect that it doesn't change the amplitude in terms of how likely it is for the superposition to collapse into that state (because the amplitude works like an absolute value), but it behaves differently during some mathematical operations. Like how a vector's length doesn't change if you negate it, but it's definitely different.

I also liked how ChatGPT used layman terms, describing the _intention_ of the process, which is to "mark" the desired state.

So the next step after marking the desired state is "inversion about the mean", which somehow leaves the desired state with an amplitude approaching 1.

**_Can you elaborate on the inversion about the mean?_**
> Certainly! The inversion about the mean is a step in Grover's algorithm that amplifies the amplitude of the desired state in the superposition while suppressing the amplitudes of the other states. This amplification is achieved by reflecting the superposition about the average amplitude of all the states.
> 
> To understand this step, consider the amplitudes of the states in the superposition after the oracle has been applied. The amplitude of the desired state is negative, while the amplitudes of the other states are positive. The goal of the inversion about the mean is to increase the amplitude of the desired state and decrease the amplitudes of the other states.
> 
> To do this, we first compute the average amplitude of all the states in the superposition:
> 
> ```
s = (1/N) * Sum(a_i), for i=1 to N
```
> where N is the number of elements in the database, and a_i is the amplitude of the i-th state in the superposition. We then reflect the superposition about this average amplitude by applying the following transformation:
> 
> ```
|ψ⟩ = -U_s * (2|ψ⟩⟨ψ| - I) * U_s|ψ⟩
```
> where U_s is the unitary operator that performs the "Hadamard transform followed by a phase flip" operation on all qubits in the superposition, and I is the identity matrix. The operator (2|ψ⟩⟨ψ| - I) is the reflection operator, which reflects the superposition about the average amplitude s.
> 
> The effect of the inversion about the mean is to increase the amplitude of the desired state and decrease the amplitudes of the other states. After a sufficient number of iterations of the oracle and inversion about the mean steps, the amplitude of the desired state approaches 1, while the amplitudes of the other states approach 0. This makes it highly likely that the desired state will be measured when a measurement is made.

I have no idea what's going on in the latter half of this response when it gets all quantum, but the general idea makes a lot of sense. If there's a magic operator that can "flip" the magnitude of the amplitudes of the states (read: lengths of the vectors) about the average of all amplitudes, then our phase flipped (read: negated) amplitude would certainly get bigger while everything else gets smaller. I can also see how repeating this a few times would exacerbate the effect.

I think I intuitively understand what Grover's algorithm is doing now. Time to go through a real-world example to make sure I'm correctly understandly what it's capable of.

**_Suppose we have a database of N=4, where each item is an integer. Grover's algorithm allows me to efficiently determine if the number 372 is in the database, is that correct?_**
> No, that is not quite correct. Grover's algorithm is not designed to find a specific value in a database directly. Instead, it is designed to find the index of a marked item in an unstructured database.

It then goes on to repeat all the steps of Grover's algorithm, which I'll omit here for the sake of brevity. ChatGPT can be quite verbose and repetitive. It also likes concluding each of its responses with a very generic summary / call-to-action which can start to feel quite patronising. But I digress.

It sounds like what I stated _is_ correct. If you can find the index of an item, then you can certainly determine if it exists or not.

**_I'm not sure I understand. Can you give me a concrete example of what Grover's algorithm can do, using an example with classical data?_**
> ...omitted for brevity...
> To use Grover's algorithm, we first encode the list of integers into a quantum superposition of states using a set of qubits. For example, if we use 3 qubits, we can encode each integer as a binary string of length 3. Then, we design an oracle that flips the phase of the superposition if it corresponds to the marked item (in this case, the number 42). The oracle can be implemented using a simple circuit that identifies the binary string corresponding to the marked item and applies a phase flip to it.
> ...omitted for brevity...

**_If we have a circuit that can detect if the integer 42 exists in a specific spot in the list, it sounds like we've solved the problem already. I don't see how an oracle can possibly exist, such that it could find an arbitrary number in a list of arbitrary numbers._**
> 
