---
title: 'Hacker101 CTF Level 5: Photo Gallery'
tags:
  - programming
  - hacking
date: 2023-06-05 23:59:16
---


## First look
![Screenshot of the level](screenshot.png)

```html
<!doctype html>
<html>
	<head>
		<title>Magical Image Gallery</title>
	</head>
	<body>
		<h1>Magical Image Gallery</h1>
<h2>Kittens</h2>
<div><div><img src="fetch?id=1" width="266" height="150"><br>Utterly adorable</div><div><img src="fetch?id=2" width="266" height="150"><br>Purrfect</div><div><img src="fetch?id=3" width="266" height="150"><br>Invisible</div><i>Space used: 0	total</i></div>

	</body>
</html>
```

The page is very basic. Notably, the 3rd image doesn't show up, and I found that `/fetch?id=3` resulted in a
`500 Internal Server Error`. Hmm.

## Flag 1
As usual, I seem to find the flags out of order.

The first thing I noticed was the source of the images: `/fetch?id=1`. The `id` parameter was an obvious place to try
SQL injection, and I was able to successfully verify that the parameter was vulnerable to these attacks by trying common
strings like single quotes and semi-colons.

In particular, I found that I received a `404` when trying to request `/fetch?id=4`, but received the image for id 1
when trying `/fetch?id=4 or 1`.

The first attack I tried was a `UNION` attack. As a quick reminder, `UNION` in SQL allows you to run multiple queries
and returns the results concatenated one after another. eg,
```sql
mysql> select 'a' union select 'b';
+---+
| a |
+---+
| a |
| b |
+---+
2 rows in set (0.00 sec)
```

So I tried `/fetch?id=4 union select 'test'`. My current mental model was hoping that the query looked something like
`select image_data from images where id = {}`, so I was hoping that I would see the string `test` in the http response.
If this was true, I would be able to run arbitrary SQL (reads, not writes) in the `UNION` and read whatever I wanted
from the database.

Instead, I just got a `500 Internal Server Error`. Even worse, there was no additional information about the crash at all.
This meant that I was going to have to perform Blind SQL Injection, which means that the only information I can glean
from each injection is "did it crash or not".

Before getting into the blind sqli (SQL Injection), I also tried _stacking queries_ with something like
`/fetch?id=1; update images set name = 'test';--` in the hopes of executing arbitrary modifications. It didn't seem to
do anything, not even crash, but I wasn't sure if it was because I guessed the table name and column name wrong, or if
the server wasn't vulnerable to stacked queries.

The key to blind sqli is to be able to produce a crash _conditionally_. You can't introduce a syntax
error, but rather need to cause an error in the query at runtime. For example, `1 / 0` produces a runtime error in some
databases.

I was pretty sure I was up against MySQL though (that's what the previous challenges used), so I would need something
like:
```sql
if(${condition},(select table_name from information_schema.tables),null)
```
The reason `(select table_name from information_schema.tables)` produces a runtime error is because a sub-select in this
context may only return a single row. `information_schema.tables` contains metadata about the database and is guaranteed
to exist, and is also guaranteed to return (much) more than a single row due to the dozens of system tables present in
every MySQL database.

I started by exploring the database schema (I still wanted to double-check my attempt on stacking queries) and wanted
to find out:
1. How many tables are there?
2. What are the names of the tables?
3. For each table, what are its columns?
4. What are the values of some of those columns?

Getting all this information via blind sqli is slow and rather painful. The general gist of the attack is to answer
a series of 'yes/no' questions and extract one bit (literally) of information at a time by asking things like:
 - is the number of tables greater than 10?
 - no? how about 5?
 - is the first character of the first table greater than 'm'?
 - no? how about 'g'?
 - what about the second character?
etc, etc

An example of getting the number of tables (formatted for readability):
```
/fetch?id=1 union select if(
  (
    select count(*) from information_schema.tables
    where table_schema not in (                       -- Exclude system tables
      'information_schema',
      'mysql',
      'performance_schema',
      'sys'
    )
  ) < 10,                                             -- Is the number of tables less than 10?
  (select table_name from information_schema.tables), -- Produces runtime error
  null                                                -- Results in a no-op basically
)`
```

An example of getting the length of a single table name:
```
/fetch?id=1 union select if(
  (
    select length(table_name) from information_schema.tables
    where table_schema not in (                       -- Exclude system tables
      'information_schema',
      'mysql',
      'performance_schema',
      'sys'
    )
    limit 1                                           -- Limit to a single table
    offset 1                                          -- Offset to get details of the second table
  ) < 10,                                             -- Is the number of tables less than 10?
  (select table_name from information_schema.tables), -- Produces runtime error
  null                                                -- Results in a no-op basically
)`
```

An example of getting a character in a string:
```
/fetch?id=1 union select if(
  (
    select ascii(                                     -- Ascii code of a character, means we can re-use common numerical binary search logic
      substring(filename, 6, 1)                       -- The 6th character of the string (SQL uses 1-indexing here)
    ) from photos
  ) < 130,                                            -- Is the ascii value of the character less than 130?
  (select table_name from information_schema.tables), -- Produces runtime error
  null                                                -- Results in a no-op basically
)`
```

I wrote a series of scripts because it would be hell-ish to do this by hand, and eventually (blind sqli is quite slow,
especially if the target service is running in a low-powered Docker container because it's a free CTF) I found:
 - There are 2 tables: `albums` and `photos`
 - `albums` has an `id` and a `title`
 - `photos` has `id`, `title`, `filename`, and `parent`
 - There is 1 album titled "Kittens" and only 3 photos, all of which are in the album
 - Photo 1: `title: Utterly adorable`, `filename: files/adorable.jpg`
 - Photo 2: `title: Purrfect`, `filename: files/purrfect.jpg`
 - Photo 3: `title: Invisible`, `filename: <flag>`

And there we have it - the file name of photo 3 is Flag 1!

## Flag 0
From exploring the database, I was able to build a better mental model of what kind of SQL query the
server was making. I suspected it was something like:

```sql
select filename from photos where id = {};
```
and then look up the file on disk to serve to the client. This explains why the photo with id 3 returns a
`500`, as well as any of my `UNION` attacks - the server would crash when it tried to look up a non-existent
file!

I tested my theory by crafting another `UNION` attack, but pointed it at a file that I knew existed:
`/fetch?id=4 union select 'files/adorable.png'` (the file for photo 1) and it worked!

Armed this with knowledge, I wondered if I had arbitrary read access to the entire filesystem of the server, and
started looking for other interesting files. I tried:
 - index.html
 - index.htm
 - index.php
 - files/\*.png
 - \*
 - ../../../etc/passwd
   - This is a file that's guaranteed to exist on Unix systems. I'm pretty sure it was a Unix system because the
     known file paths used forward slashes rather than backslashes. Although the code could just be converting it
     into platform-specific paths.
 - /etc/passwod
 - windows-specific .ini file that I can't exactly recall now
 - encoding the forward slashes as `%2e` (pretty sure I didn't need this because the know file paths worked)

...and nothing worked. Everything resulted in a `500`.

I was stumped here for a long time, until I got the following hint for this level:

> This application runs on the uwsgi-nginx-flask-docker image

Some very quick research led me to look for `main.py` by using `/fetch?id=4 union select 'main.py'` and it immediately
worked. Being able to see the actual source code was like turning on the light - instead of blindly groping around and
crafting theories, I had perfect knowledge of the target I was trying to exploit.

The source code immediately answered a bunch of questions that I had:
 - The exact SQL query powering the `fetch` endpoint: `'SELECT filename FROM photos WHERE id=%s' % request.args['id']`
 - Why `..` didn't work in my directory traversal attack: `return file('./%s' % cur.fetchone()[0].replace('..', ''), 'rb').read()`

It also revealed a few other very interesting things:
 - Flag 0 :)
 - Database connection credentials: `MySQLdb.connect(host="localhost", user="root", password="", db="level5")`
 - Shell injection vulnerability:
```python
rep += '<i>Space used: ' + subprocess.check_output('du -ch %s || exit 0' % ' '.join('files/' + fn for fn in fns), shell=True, stderr=subprocess.STDOUT).strip().rsplit('\n', 1)[-1] + '</i>'
```

## Flag 2
The `du -ch` command attempts to calculate the size of files on disk.
 - `-c` Prints a total size of all specified files
 - `-h` Prints sizes in human-readable form (eg, 8.0K instead of 8192000)

The final command on the shell would look like `du -ch files/adorable.jpg files/purrfect.jpg files/<flag1> || exit 0`
which explains why the webpage displays "Space used 0" - the non-existent `files/<flag1>` file is causing the command to
error out.

After running the shell command, the Python code also only reads the last line of `stdout` (`.rsplit('\n', 1)[-1]`),
with the intent of only printing out the total size.

There is an obvious exploit: inject an alternate shell command by modifying the filename to `; pwd`, for example, which
would make the entire command `du -ch files/adorable.jpg files/purrfect.jpg files/; pwd || exit 0` and render the output
of `pwd` (the current directory).

But how do we modify the filename of one of the files? I thought I could use the database credentials found in the source
code to connect directly to the database, but the port wasn't open to the internet.

I was again stumped, until this hint:

> Stacked queries rarely work. But when they do, make absolutely sure that you're committed

I surmised that adding the SQL `COMMIT` statement would allow the stacked queries to work - and it did!

The source code shows that the `MySQLdb` python library is used to execute database queries, and a bit of research
shows that it executes queries in a database transaction automatically, and the `autocommit` option is set to false
by default. The library expects the user to use the `commit()` function to commit the transaction, but we can get
away with adding it to the end of our stacked queries.

```
/fetch?id=1; update photos set filename = ';pwd' where id = 3; commit;--
```

This payload effectively dumps the output of `pwd` straight into the HTML when we view the home page, where the
original space calculation output was.

One last thing to note is that the Python server only returns the last line of shell output, so I did a quick replacement
of newlines in order to get all of the output. eg, `; cat /etc/passwd | tr "\n" "|"` which replaces all newlines with
pipe characters.

I tried a variety of shell commands and it was _so liberating_ to get all the output reflected back to me. Compared to
the blind sqli from before, I felt like a starving man who stumbled into a feast. I basically had a full shell on the
server (and even briefly considered writing a script to emulate a CLI) and could probably do all sorts of nasty things.

I had fun exploring the filesystem and looking at all the files, but eventually find the inspiration to look at the
environment variables with the `env` command.

Lo and behold, there I found all 3 flags.

## Summary
**Time taken:** ~5 hours

**Hints needed:**
 - That it was a Flask webserver
 - That `commit` could make stacked queries work

## Takeaways
 - SQLMap automatically dumps databases with blind sqli (but I found my experience very educational)
 - Knowing your target environment (backend framework, OS, DB, etc) is _extremely_ useful
 - Broad knowledge about many different environments is crucial to be a successful hacker
 - Try `COMMIT` for stacked queries

## References and links
 - https://portswigger.net/web-security/sql-injection/cheat-sheet
 - https://sqlmap.org/