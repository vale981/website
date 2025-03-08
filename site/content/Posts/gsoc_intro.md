+++
title = "KDE GSOC: Intro"
author = ["Valentin Boettcher"]
date = 2021-06-27T15:00:00-04:00
tags = ["GSOC"]
categories = ["KDE"]
draft = false
+++

Hi folks, talking to you over the interwebs is Valentin Boettcher who
is overhauling the Deep Sky Object (DSO) system in the KStars Desktop
Planetarium for the Google Summer of Code anno domini 2021.

This is the first post in a series and rather late in the coming, so
let's get right to it.

I'm currently studying for a master’s degree in physics at the TU-Dresden
in, you've guessed it correctly, the beautiful city of Dresden
(Germany). In Germany, we do have two study terms per year and the
summer term usually coincides neatly with the GSOC so that I couldn't
participate in past years. This time around however, my schedule was
finally sparse enough for me to have a go at it, and here we are :).

My first contact with KStars development was back in 2017 while I
spent a year in New Zealand and had a lot of time at hand. My
reasoning was, that I could learn mathematics and physics in UNI and
should funnel my enthusiasm into familiarizing myself with software
development and the open source software community. I promptly wiped
my hackintosh laptop to put Linux with KDE on it[^3]. After reading
ESR's famous ["How To Become A
Hacker"](<http://www.catb.org/~esr/faqs/hacker-howto.html>), I followed
the advice given therein, which was to find an open source project and
start hacking on it. I already liked KDE and space, so KStars was in
the center of the Venn-diagram :P.  I went ahead and busied myself
with one of the junior jobs listed on the KStars web-site[^2]. I
quickly found that I liked figuring out how stuff in KStars worked and
also got in contact with my mentor Jasem Mutlaq who was always
available to answer questions and endure my barrage of instant
messages on matrix :P. My second job was to draw comets a tail and
learned that it is wise to do some code archaeology before going ahead
and implementing functionality that is already present. From there on
I contributed more or less regularly when I found the time in my
semester breaks.

Now, finally, let's talk a wee bit about the actual GSOC project.  In
KStars, everything that isn't a Star or an object in our solar system,
an asteroid, a satellite or a comet (I'm sure I forgot something) is a
deep sky object (DSO). Prominent members of the DSO caste are galaxies
(think M31, Andromeda), asterisms and nebulae. Of course there are a
plethora of catalogs for specific types of DSOs (for example, Lynds
Catalog of Dark Nebulae) as well as compilations like the New General
Catalogue.  The system for handling those catalogs in KStars has grown
rather "organically" and is now a tangle between databases, CSV files
and special case implementations. Many catalogs were mentioned
explicitly in the code, making it hard to extend and generalize. Also,
the sources of the catalogs and methods how they were transformed into
the KStars format were inhomogeneous and hard to reproduce, making
deduplication almost impossible. Finally, KStars just loaded all the
DSOs into memory and computed their position on the virtual sky for
every draw cycle, which made all too large catalogs infeasible.  My
task is now (and has been since the beginning of June) to implement a
unified catalog format which can be loaded into a central database and
supports deduplication. Furthermore, taking inspiration from the
handling of star catalogs in KStars, the objects should be trixel[^1]
indexed and cached in and out of memory (but only for large
catalogs). Finally, it would be very desirable to make the
creation/compilation of the catalogs reproducible and easily
extendable to facilitate future maintenance.

This sounds like a big heap of stuff to get done and in the next post
I will be detailing how it's going so far :).

Cheers,
Valentin

[^1]: In KStars the sky is subdivided into triangular pixels "Trixels".

Assigning each object to a trixel makes it efficient to retrieve all objects from a certain part of the sky.

[^2]: which had to do with figuring out why some faint asteroids where missing

[^3]: which I knew from my school time when I used it on my netbook because there was a cool neon "Hacker" theme for it :P
