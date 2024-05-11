+++
title = "KDE GSOC: Second Coding Period; Some Notes on the Catalog Repo."
author = ["Valentin Boettcher"]
date = 2021-08-08T12:15:00-04:00
tags = ["GSOC"]
categories = ["KDE"]
draft = false
+++

> TL;DR DSO catalogs in KStars are now generated reproducibly in the
> CI. A list of available catalogs and documentation can be found [here](https://protagon.space/catalogs/pages/catalogs.html).

As promised [last time]({{< relref "gsoc_1" >}}) I'll now go a little into the [Catalogs
Repository](https://invent.kde.org/vboettcher/kstars-catalogs).

Usually DSO catalogs are pretty static and rarely change due to the
nature of their contents. But although galaxies do not tend to jump
around in the sky, catalogs still get updates to correct typos or
update coordinates with more precise measurement. Our primary catalog
[OpenNGC](https://github.com/mattiaverga/OpenNGC) for example gets updates quite regularly.

{{< figure src="/images/GSOC:_Second_Coding_Period;_Some_Notes_on_the_Catalog_Repo./2021-08-08_12-21-27_screenshot.png" caption="<span class=\"figure-number\">Figure 1: </span>OpenNGC is being updated regularly." >}}

And even though a catalog might not change, it would nevertheless be
desirable to have a record on how it was derived from its original
format in a _reproducible_ way[^fn:1]. Last but not least, having all catalogs in a
central place in kind of the same format would make deduplication a
lot easier.

The question is: how does one define a convenient yet flexible format
that nevertheless enforces some kind of structure? My answer was: with
some kind of package definition. What about the flexibility part?
Well, basically every catalog is just a python module that must
implement a class. By overwriting certain methods, the catalog can be
built up. The framework provides certain support functionality and an
interface to some catalog database features by way of a python binding
to some `KStars` code. Apart from that one has complete freedom in
implementing the details although some conventions should be
followed[^fn:2].

A simple random catalog looks like the following listing.

```python
def generate_random_string(str_size, allowed_chars=string.ascii_letters):
    return "".join(random.choice(allowed_chars) for x in range(str_size))


class RandomCatalogBase(Factory):
    SIZE = 100
    meta = Catalog(
        id=999,
        name="random",
        maintainer="Valentin Boettcher <hiro@protagon.space>",
        license="DWYW Do what ever you want with it!",
        description="A huge catalog of random DSOs",
        precedence=1,
        version=1,
    )

    def load_objects(self):
        for _ in range(self.SIZE):
            ob_type = random.choice(
                [ObjectType.STAR, ObjectType.GALAXY, ObjectType.GASEOUS_NEBULA]
            )
            ra = random.uniform(0, 360)
            dec = random.uniform(-90, 90)
            mag = random.uniform(4, 16)
            name = generate_random_string(5)
            long_name = generate_random_string(10)

            yield self._make_catalog_object(
                type=ob_type,
                ra=ra,
                dec=dec,
                magnitude=mag,
                name=name,
                long_name=long_name,
                position_angle=random.uniform(0, 180),
```

It implements only the `load_objects` build phase and is a kind of
minimum viable catalog.

The basic idea behind the structure of a catalog implementation is
that the build process can be subdivided into four _phases_ which can
be partially parallelized by the framework.

In the download phase each catalog defines how its content may be
retrieved from the Internet or otherwise acquired. In the load/parse
phase the acquired original data is being parsed and handed over to
the framework which takes care of molding it into the correct
format. During the deduplication phase each catalog can query the
catalog database to detect and flag duplicates. And in the final dump
phase the contents of each catalog are written into separate files
which `KStars` can then import[^fn:3].

If you are interested in the details I can recommend the [documentation](https://protagon.space/catalogs/)
for the catalog repository.

After implementing the framework porting over all the existing
catalogs to the new system, I went on to configure the KDE Invent CI
to rebuild the catalogs upon changes. The CI artifacts are sync-ed to
the `KNewStuff` data server for KStars periodically and users are able
to update their catalogs to the latest version.

To get the CI working I had to create a [Docker image](https://invent.kde.org/vboettcher/python-kstars-docker) that encapsulates
the more or less complicated build process for the KStars python
bindings. This container is updated weekly by CI and is also suitable
as a quick-and-easy development environment for new catalogs.

That's it for today but do not fret. This is not all that I've
done. There's still more to come including something that has to do
with the following picture.

{{< figure src="/images/GSOC:_Second_Coding_Period;_Some_Notes_on_the_Catalog_Repo./2021-08-08_13-09-43_screenshot.png" >}}

Cheers,
Valentin

[^fn:1]: And in a way that hopefully lasts
    for some time. Currently very few people know how to generate KStars'
    deep star catalogs...
[^fn:2]: I haven't yet worked those out yet TBH.
[^fn:3]: The catalog package files actually
    do have the same format as the main DSO database :).