+++
title = "Modern Python REPL in Emacs using VTerm"
author = ["Valentin Boettcher"]
date = 2024-05-11T19:47:00-04:00
tags = ["emacs", "python"]
categories = ["Hacks"]
draft = false
+++

As alluded to in [Poetry2Nix Development Flake with Matplotlib GTK
Support]({{< relref "poetry2nix-development-flake-with-matplotlib-gtk-support" >}}), I'm currently in the process of getting my "new" python
workflow up to speed. My second problem, after dependency and
environment management, was that fancy REPLs like [ipython](https://ipython.org/) or [ptpython](https://github.com/prompt-toolkit/ptpython.git)
don't jazz well with the standard `comint` based inferior python repl
that comes with `python-mode`. One can basically only run ipython with
the `--simple-prompt` flag which removes features like
syntax-highlighting and auto-completion. Especially annoying is, that
only the `tkinter` backend for `matplotlib` works in this mode.

The package `elpy` comes with some improvements, especially when it
comes to sending part of a buffer to the repl, but it comes with all
sorts of baggage that interfere with my emacs setup.

From my jolly [Julia](https://julialang.org/) days I'm used to [julia-vterm](https://github.com/shg/julia-vterm.el). This emacs package
runs a Julia REPL using a full terminal emulator ([emacs-libvterm](https://github.com/akermu/emacs-libvterm)). So
in the pursuit of a nice hack, I `M-x replace-string`'d the word `julia`
with `python` and gave it a shot. Remarkably, the whole thing just
worked without much tweaking and you can enjoy the result by checking
out the [GitHub repo](https://github.com/vale981/python-vterm.el).

The idea of extending the original `julia-vterm` package to support
python as well is not without elegance. However, the code base is not
too large and -- owing to the differing sensibilities of the julia and
python communities -- the feature-set is likely to diverge in the
future.

{{< figure src="/home/hiro/Documents/Projects/website_new/2024-05-11_20-12-52_screenshot.png" caption="<span class=\"figure-number\">Figure 1: </span>And this is the result!" >}}
