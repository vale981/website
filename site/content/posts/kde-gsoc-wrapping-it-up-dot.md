+++
title = "KDE GSOC: Wrapping it up…"
author = ["Valentin Boettcher"]
date = 2021-08-16T18:53:00+02:00
tags = ["GSOC"]
categories = ["KDE"]
draft = false
+++

Well, we all know that the work on open source projects is never truly
finished, but all of the core goals have been achieved and the time
is up :). In this post I'll briefly summarize my GSOC work and then
talk about one last small but user-facing feature that I've
implemented.

I've successfully implemented a new DSO backend and smoothed out most
of the bugs. The [python framework](posts/.org) does work satisfactory and all
existing catalogs have been ported. There remains the UGC catalog
which will be imported in the future, either by me or by another
member of the project. The latter option would be a good way to
battle-test the documentation and I would prefer this option because I
do not want to remain the only person familiar with the system.

To quantify my contributions during the GSOC period see the snippet
below, although I do not think such numbers have much to say[^fn:1].

```shell
Valentin Boettcher <hiro@protagon.space>:
       insertions:    15193  (19%)
       deletions:     23402  (35%)
       files:         312    (21%)
       commits:       76     (23%)
       lines changed: 38595  (26%)
```

Furthermore there is [the list of my merge requests](https://invent.kde.org/education/kstars/-/merge%5Frequests?scope=all&state=merged&author%5Fusername=vboettcher) which does go into
more detail.

The user-facing side of my work is not very prominent. There is a
small GUI for managing catalogs that allows importing, exporting,
creating and editing catalogs.

{{< figure src="/images/KDE_GSOC:_Wrapping_it_up.../2021-08-16_20-13-41_screenshot.png" >}}

{{< figure src="/images/KDE_GSOC:_Wrapping_it_up.../2021-08-16_20-18-49_screenshot.png" >}}

There is also a basic CSV importer that should make it easier for
users to get their own custom data into KStars.

{{< figure src="/images/KDE_GSOC:_Wrapping_it_up.../2021-08-16_20-30-54_csv_openngc.png" caption="Figure 1: The CSV importer. It sure needs some prettying up :P." >}}

Nevertheless, the main goal of my work was to create a seamless
replacement for the old DSO system of which the user should not be too
aware. To that end, I've implemented a feature that should have been
in my overhaul from the beginning: a mechanism to import custom
objects [from the old DSO database](https://invent.kde.org/education/kstars/-/merge%5Frequests/377). Now, on startup the user is being
asked whether the old database should be imported if it is present.

{{< figure src="/images/KDE_GSOC:_Wrapping_it_up.../2021-08-16_20-38-10_screenshot.png" >}}

And finally: Colors!

The DSOs always had a distinct color depending on the catalog they're
from. Right from the outset one complaint from early testers were the
garish colors that I chose for the catalogs. I "fixed" this problem by
simply choosing more subdued colors. But colors are a matter of
personal taste. Also, a single color can't fit all of KStars' color
schemes. Therefore colors can now be customized for each catalog and
color scheme through a "pretty" dialog.

{{< figure src="/images/KDE_GSOC:_Wrapping_it_up.../2021-08-16_20-52-46_screenshot.png" caption="Figure 2: The \"pretty\" color picker." >}}

Now you can do things like this:

{{< figure src="/images/KDE_GSOC:_Wrapping_it_up.../2021-08-16_20-49-03_screenshot.png" caption="Figure 3: Color Scheme: Moonless Night" >}}

{{< figure src="/images/KDE_GSOC:_Wrapping_it_up.../2021-08-16_20-51-16_screenshot.png" caption="Figure 4: Color Scheme: Starchart" >}}

And again I've learned that user feedback is very important. I would
never have thought of this feature on my own but must admit that it
enhances the usability of KStars greatly.

With that oddly specific foray into the world of colors I now conclude
this blog post and thank you for your attention.

Cheers,
Valentin

[^fn:1]: I deleted the old OpenNGC text catalog which contained more than ten thousand lines :P.