+++
title = "Neuseeland Restauration"
author = ["Valentin Boettcher"]
date = 2021-08-03T14:53:00+02:00
categories = ["Uncategorized"]
draft = false
+++

TL;DR Die Neuseeland-Blogposts sind Und damit koennt ihr nun unter
[wieder da](/categories/neuseeland) und korrekt datiert.

Zwar lagen mir die Neuseeland-Blog posts als `markdown` quelle vor,
jedoch hatte ich den Erstellungszeitpunk unvorteilhafter Weise aus den
Dateisystem-Metadaten[^fn:1] ausgelesen. Nach der Neuinstallation meines
servers vor ein paar Jahren waren diese Metadaten vollends verloren.

Meine erste Idee war zu versuchen die Zeitpunkte anhand der Email
Newsletter zu rekonstruieren. Auf diesem Wege versprach ich mir jedoch
keinen baldigen Erfolg da mein eigenes Email Archiv nicht soweit
zurueckreicht.

Zu meiner grossen Freude war letztendlich gar keine aufwaendige
Archeologie notwendig, da ich damals ausversehen einmal meinen Blog
Index in `git` eingecheckt hatte.

```shell
$ git show b1cf78b7182f0364343d2a87a1b361e7dc833688^1:data/indexes/post.json
```

Und schon hattte ich die ganze Suppe (sogar vollstaendig) in einem
maschinenlesbaren Format.

Ein wenig python verwandelte das Ganze in das neue Blog format.

```python
import sys
import json
import datetime

with open(sys.argv[1], "r") as f:
    data = json.load(f)

    for post in data:
        export_name = (post["content"]).split("/")[-1][:-5]
        with open(post["content"][1:], "r") as cont:
            content = cont.read()
        date = datetime.datetime.strptime(post["created"], "%Y-%m-%dT%H:%M:%S.%fZ")
        print(
            f"""
*** {post["title"]}
:PROPERTIES:
:EXPORT_FILE_NAME: {export_name}
:EXPORT_DATE: [{date}]
:END:
{content}""")
```

Und damit koennt ihr nun unter [Neuseeland](/categories/neuseeland) die alten posts lesen.

[^fn:1]: `ctime`, creation time