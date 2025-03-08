+++
title = "Hours of my life"
author = ["Valentin Boettcher"]
date = 2025-01-10T03:57:00-05:00
categories = ["Tricks"]
draft = false
+++

have been wasted why the line spacing and `DIV` calculation (typearea) of KOMA script was hanging with `LuaTeX`.

TL;DR: `LuaTeX` and `setspace` don't go together.

I stripped out the custom fonts, I removed (seemingly) every package, I ported my document to `PDFTex`. At long last after HOURS spent over several days if found the culprit. This time, I actually removed _everything_ and there it was: `\usepackage{setspace}` was the culprit. Why did I include it? I don't anymore and I wont spend any more time on this.
