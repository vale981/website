+++
title = "Kindle Scribe Sync"
author = ["Valentin Boettcher"]
date = 2024-11-17T12:13:00-05:00
tags = ["org", "roam", "emacs", "python"]
categories = ["Hacks"]
draft = false
+++

TL;DR check out <https://github.com/vale981/kindle_fetch>.

I "recently" treated myself to a Kindle Scribe e-ink tablet which is
somewhat like a ReMarkable[^fn:1] but on the cheap. There is no free
lunch however, and price you pay is a somewhat sub-par software. One
quite essential missing feature is convenient synchronization with
other devices.  As I use the tablet mainly for calculations, I rely on
having the previous pages available at a glance (just as they would be
if I were using paper). Furthermore, I would like to integrate the
handwritten notes into my [org-roam](https://www.orgroam.com/) note taking system.

One solution would be to jailbreak the kindle, ssh into it and
automatically download a notebook when it changes. The conversion to
PDF could then be handled by the [Calibre KFX input plugin](https://www.mobileread.com/forums/showthread.php?t=291290). However,
that ship has sailed for me as the firmware on my Kindle is too new.

This led me to come up with the following [bodge](https://www.youtube.com/watch?v=lIFE7h3m40U). Amazon offers a
service that converts a notebook to PDF and sends it to your email
upon request and this we can exploit to our heart's content. I wrote a
[script](https://github.com/vale981/kindle_fetch) which monitors your email inbox via IMAP[^fn:2] using [aioimaplib](https://github.com/bamthomas/aioimaplib)
and detects incoming emails from Amazon containing download links for
the exported notebooks. Once such an email is detected, the associated
PDF is downloaded into a directory on the local file system
(`~/kindle_dump` on my machine). The name is inferred from the contents
of the email, although at the moment this doesn't include any hint
about which folder in the directory structure on the Kindle it came
from. The email is then conveniently deleted so as to not clutter your
inbox[^fn:3].

Additionally, the latest downloaded PDF is copied into to a file of
your choice (`~/kindle_dump/.latest` in my case). I then run `zathura
~/kindle_dump/.latest` which displays that PDF on my large 4K monitor
and auto refreshes each time a new Kindle email comes in. I also
configured Zathura to show four pages per row (`set pages-per-row 4`) so
that I can see eight pages at a glance (see figure below).

{{< figure src="/ox-hugo/screen:d22fec54-83c6-4cc6-a701-c29270868804.png" caption="<span class=\"figure-number\">Figure 1: </span>An example of what my ugly handwriting looks like :)," >}}

Additionally, the following elisp snippet allows me to attach the
latest note to an org-roam node with a couple of keystrokes.

```elisp
(defun attach-kindle-note (text)
  "Attaches the latest kindle note to the current org buffer and links to it with TEXT."
  (interactive "sText: ")
  (let* ((name (org-id-new "kindle"))
         (tmp (format "/tmp/%s.pdf" name))
         (file "/home/hiro/kindle_dump/.latest.pdf")
         (text (if (string-empty-p text)
                   (format "handwritten note %s" (current-time-string)) text)))
    (copy-file file tmp t)
    (org-attach-attach tmp nil 'mv)
    (insert (format "[[attachment:%s.pdf][%s]]" name text))))
```

The integration into my note-taking system is the most important part
the whole bodge for me.

Annoyingly, Amazon has already changed the email format twice since I
initially hacked together this solution. However, each time I come up
with a more robust way to get the PDF download link and one's free
time has to be spent in some way or another anyways.

And if you're reading this sentence, you've made it to the end of this
rather redundant post. Thank you for reading. I hope that at least
some people find this bodge useful. If there's sufficient interest, I
might add OAuth2 support.

On the road-map for future (think of years, not months) posts are my
mu4e/mbsync setup with OAuth support on NixOS, my Xournal++ setup with
vim-like keybindings and a patch to make the output files more
compatible with version management and finally, my org-roam braindump.

In other news, my fork of [julia-vterm](https://github.com/vale981/julia-vterm.el) is now called
[py-vterm-interaction](https://github.com/vale981/py-vterm-interaction.el) and available on MELPA.

[^fn:1]: Years ago, I sold my ReMarkable in to buy a cheap drum-set and used a
    graphics tablet instead. I have a pretty neat set-up for that, too but
    that's a tale for another time.
[^fn:2]: Unfortunately, OAuth is not supported, so GMail/Outlook won't work out
    of the box.
[^fn:3]: I additionally have a Sieve filter running on my server that
    automatically sorts the Kindle emails into a specific folder, although
    that is not strictly necessary anymore. You might want to setup a
    separate email account for this purpose altogether if you're not
    comfortable with having a (potentially buggy) python script mucking
    around in your emails.
