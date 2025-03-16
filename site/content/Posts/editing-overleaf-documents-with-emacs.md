+++
title = "Editing Overleaf Documents with Emacs"
author = ["Valentin Boettcher"]
date = 2025-03-15T21:24:00-04:00
tags = ["emacs", "LaTeX"]
categories = ["Hacks"]
draft = false
+++

I am _still_ in the process of writing a paper. As we're now in the
editing stage, my supervisor has requested that I edit in Overleaf
directly, so that he can see my changes through the "tracked changes"
feature of overleaf. Weirdly, the git integration does not support
that feature and so I was left with having to leave emacs.

But, no! I'd rather spend ten hours to hack together an emacs
integration for overleaf than spending just one minute in the overleaf
editor. (Well this sentiment is a bit tongue-in-cheek, but there are
many things that make me want to stay in emacs.)

{{< figure src="/ox-hugo/demo.gif" >}}

This feat was accomplished by reverse engineering the old version of
[socket.io](https://socket.io/) overleaf useses and prodigious use of the network analyzer
of Firefox. The strangest issue I encountered was the weird way
Overleaf encodes unicode (too many bits per character, seems to be a
MySQL thing). The stopgap solution I found is to actually call out to
`node.js` and use the same bit of code that overleaf uses to decode:

```js
process.stdout.write(decodeURIComponent(escape(process.argv[2])));
```

Anyhow, I wanted to write way more than this but as always I am too
lazy! Check out the [Git](https://github.com/vale981/overleaf-connection.el) repo :).
