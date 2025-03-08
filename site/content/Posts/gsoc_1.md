+++
title = "KDE GSOC: Community Bonding and First Coding Period (May 17 - July 11)"
author = ["Valentin Boettcher"]
date = 2021-07-11T15:00:00-04:00
tags = ["GSOC"]
categories = ["KDE"]
draft = false
+++

Of course the task I described in the looks and is quite
monumental. That is why I laid some of the groundwork for my GSOC
beforehand (in the actual German semester breaks). This work continued
in the community bonding and first coding period and will therefore be
described here.

But first I want to thank my mentor Jasem Mutlaq for his support, his
patience with me and his nerves of steel. My mood levels were somewhat
similar to a huge-amplitude sine wave those last weeks.

Now to the meat...

I began by studying the existing deep sky object implementation in
KStars to identify what structure the new catalogs should have and
what the smallest irreducible core of functionality was I could
replace to make integration easier. I discovered that the catalogs
were a mix of SQL databases and text files, somehow loaded at startup
and then appended to some linked list. There was some deduplication
implemented but like most DSO code it was oddly catalog
specific. Especially the Messier, IC and NGC catalogs were often
mentioned in the code. Also the explicit distinction between stars and
DSOs made writing general code complicated but I found a consistent
set of data fields shared by all catalog objects which all admitted
sane defaults. It wasn't bad code or anything like that. Just the
product of "organic groth" with many thing I wanted already present in
some way but somewhat all over the place. I admit that I studied the
code just enough to find out what exactly I had to replace and maybe I
could have reused more of the existing code but I've picked this
specific path in the multiverse, so let's get on with it. Just a shout
out to all who did previous work on the DSO code among whom are, just to
name a few, Jason Harris, Rishab Arora, Thomas Kabelmann and Akarsh
Simha.

With this knowledge I was able to go forward and devise a concrete
plan for implementing the new DSO system. First of all, albeit I would
love to use `std::variant` and some kind of entity component system
for the different DSO types I settled with a one-for-all type for deep
sky objects. The primary reason for this was, that KStars uses `C++14`
which lacks variants (and the extremely useful
`std::optional`). Furthermore the DSOs all share common structure, so
this was just the simpler and thus preferable option. The second
design decision was not to load all of the DSOs into memory, but
instead to take inspiration from the deep star catalogs. For one they
are dynamically loaded from a special trixel indexed format so this
already was within the formulated goals of the endeavor. On the other
hand the notion of having "canonical" copies of catalog objects in
memory and syncing their mutation with the database system seemed
overly complicated. The catalog database should be the single source
of truth and not the (ephemeral) memory of KStars.

When a specific object is needed, it should just be retrieved from the
database locally in the code instead of searching some in memory list
in KStars or shooting around with pointers. This notion is somewhat at
odds with how things were and are done in KStars which created some
interesting problems later on as we shall see. These ideas somewhat
dictated the rest of the plan which I (for the first time in my
programming career) completely wrote down in advance. The heart of it
all is the database manager which abstracts maintaining, reading from
and writing to the database. As always one should justify the creation
of a special data type. In this case it was the requirement that the
database access should be painless and could be handled locally
anywhere in the code just by creating another instance of the database
manager. The manager should handle retrieving objects and catalog meta
information as well as importing, editing and exporting catalogs.

The structure of the database itself was another point of
consideration. Naturally each catalog should have its own table. But
how should deduplication work?  The method I settled on is really
quite simple. Each object gets a (relatively stable) hash that is
calculated from some of its properties which is henceforth called the
ID.  When two objects (from different catalogs or otherwise) are the
same <span class="underline">physical</span> object, then they will both be assigned the same
object id (OID) which is just the ID of the object in the "oldest"
catalog (with the lowest catalog id), trying to make it stable under
the introduction of new catalogs. Additionally each catalog is
assigned a priority value which is just a real number (conventionally
between zero and one). When loading objects from the database into
KStars and there are multiple objects with the same OID only the one
from the catalog with the highest priority will be loaded. This simple
mechanism should cover the requirements of KStars quite well and is
relatively easy to implement.

There I ran into an issue that demanded some research and
table in the database.  The simplest option would be just to create a
benchmarking. Remember that each catalog is represented by its own
so-called view, a dynamic "virtual" table that combines all the
catalog into one homogeneous table. SQL magic could automatically
perform the deduplication algorithm outline above and everything would
be fine and dandy. However, benchmarking revealed that actually
writing the view into its own table, henceforth called the master
catalog/table, increased the performance quite considerably, enough so
to justify the increased complexity in the implementation. And then I
discovered SQL indexes. A gift from the heavens! They increased
performance on loading objects in a trixel from the master catalog
roughly threefold and I was sold on the master catalog approach. So to
summarize it all; a deduplicated view of the combined catalog tables
is being created and then written into the master table. This has to
be done for every modification of the catalogs but is relatively fast
(just not fast enough to be done 20 times per second). Later
experimentation showed that this approach could accommodate catalogs
up to a million objects in size.

I also created a catalog file format, which is just an sqlite database
file with the application id set to a special value with almost the
same structure as the catalog database proper. The application id
enables KStars to check if the database is really a catalog file and
not to rely just on the structure of the contained database for
that. In the future the `file` command and other utilities like file
managers could be made aware of this special application id to
recognize the catalog files. We will leave it this level of detail for
now. For more details please refer to my [notes](https://protagon.space/stuff/kstars_cleaned.org).

Of course the operations on catalogs have to somehow be accessible in
the GUI of KStars so this was another point of action. Before that
however the glue between the database manager and the usual sky
composite system had to be implemented. In KStars different types of
objects (Stars, Comets, Asteroids, etc. pp.) all are implemented as
components with a unified interface. These components provide methods
for loading and drawing objects, as well as utilities to find objects
near a certain point on the sky and similar things. The loading and
drawing part was relatively simple to implement. The drawing code
could be straight up reused from the old implementation and the
loading was essentially covered by the database manager but with a
twist.

To support very large catalogs it would be desirable to only have
objects in memory which are currently visible. Thus a LRU cache was
implemented with the trixel id, which essentially labels a portion of
the sky, as key. This cache is fully unit tested and relies completely
on standard library containers so not a single pointer appears in the
code.[^fn:1] As an added bonus, the cache is completely transparent by
default and only takes effect if configured to so and therefore
includes the typical use case of comparatively small catalogs up to
ten-thousands of objects.

But here the culture clash between the new DSO implementation and the
traditional KStars way of things became apparent. In many places
KStars expects pointers to so called `SkyObjects` with no real clue as
to where they are actually stored and how their memory is managed and
with the implication that the object is expected to live
forever. Well, the DSOs from the catalogs aren't supposed to be kept
around forever and thus a compromise is in order. So whenever a
pointer to an object is required, it is inserted into a linked
list[^fn:2] in a hash table with the trixel as index or is taken from
there if it's already present. I hope that we can eventually
transition away from raw pointers and manage life time either
explicitly or with smart pointers.

With this done and basic drawing working I went on to implement a
basic GUI for catalog management[^fn:3].  I also wrote unit tests for
the database functionality which proved itself as very useful later
on. After that I couldn't delay anymore. Back when I implemented the
component for the new DSOs I went as far as getting it to compile and
not much further[^fn:4].

Now I had to go around and find out what broke. A lot broke and I did
not find all of it until the big merge :P.  A rather interesting
source of work happened to be the way metadata like observation notes
and image links were stored. They came from a text file and then were
loaded into the sky objects at startup and somehow synchronized with
the text files on mutation. This, of course, played not well with the
new DSOs as they were ephemeral. So I replaced the whole shebang with
a hash table which incidentally improved startup performance. The rest
of the integration work was similarly interesting and continues
today. I will not go into it further but feel free to look at the
KStars commit history.

Just yesterday I added a feature back in that I had axed accidentally
to the dismay of its original author. That showed me that I am not
entitled to judge the merit of individual features and whether they
could be sacrificed for the "greater good". The answer is: They
cannot! Another lesson I've learned is, that too much magic just ain't
no good. I had created a variadic template wrapper for the `QSqlQuery`
type for syntactical convenience and shot myself in the food with
it. It ended up obscuring an error message and prevented me from
reproducing a crash that users on certain platforms were
experiencing. After a not-so-great couple of days I, with the help of
two kind people, finally found the lowest common denominator of the
problem: an old, but still supported version of QT which bundled an
old version of sqlite which in turn did not support the `NULLS FIRST`
directive that I was using. Turtles all the way down. Although I
tested all my changes on KDE Neon (I am on NixOS primarily) the wise
thing would have been to develop or at least test everything with an
older QT version from the get go.

Also, although I had put in version checking into the database code, I
didn't provide a mechanism for upgrading the database format to new
versions. This I now remedied by introducing a simple mechanism that
applies database modifications successively for each version
upgrade. So if I go from version two to version four it will be
upgraded from version two to three and then to four which I understand
is the way those things are usually done.

Now, I did do at least some "constructive" work, adding a (admittedly
ugly) CSV importer so that users can import arbitrary CSV-ish
catalogs. The greater chunk however I will cover next week: The python
catalog package tooling with continuous integration and
deduplication. The catalogs churned out by that framework are then
installed via the `KNewStuff` framework. I discovered two interesting
bugs in this framework because KStars seems to be almost the only
program using the framework in this specific way.

If you made it this far, I applaud and thank you for your endurance.
See you next time.

Cheers,
Valentin

P.S. Currently I am working on documenting both the new DSO GUI and
the python tooling. I hope eventually they will pass the "noob test"
:P. But, as you may have recognized above, I am not the best explainer.

[^fn:1]: As a matter of fact, I set out with the goal not to do any
    manual memory management and not to use a single pointer in the new
    code. I have been successful thus far if you would be so lenient not
    to count glue code for legacy KStars systems.
[^fn:2]: References to objects in linked lists are stable.
[^fn:3]: See the KStars Handbook.
[^fn:4]: I really appreciate c++ as a compiled language.
