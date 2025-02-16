+++
title = "Subfigures in RevTex"
author = ["Valentin Boettcher"]
date = 2025-02-15T19:32:00-05:00
tags = ["ATTACH"]
categories = ["Hacks"]
draft = false
+++

I'm in the process of writing a paper at the moment (yay!). Merrily I
add a formula here a plot there; but my supervisor zeroes in
immediately on a two-panel figure with sub figures called "a" and "b"
saying:

> The sub figure labels go in the top-left corner. You've read papers
> before, haven't you.

Having indeed read papers before, I must have neglected to pay proper
attention to sub-figure labels -- an inexcusable blunder -- so this
was news to me. Unwilling to fix the issues by monkeying around in
inkscape, I turned to stack overflow. We have the pleasure to be using
the [RevTeX](https://www.ctan.org/tex-archive/macros/latex/contrib/revtex) document class and annoyingly this class happens to be
incompatible with both the [subcaption](https://ctan.org/pkg/subcaption?lang=en) package and the part of [subfloat](https://ctan.org/pkg/subfloat)
that controls where the captions go. The aforementioned packages are
"the way" to do sub figures and most other solutions I found involved
monkeying around with minipages.

So that's what I did; but I wanted to do properly so I'd be able to
give the sub figures referenceable labels. I came up with a
surprisingly short piece of hacked-together LaTeX package which we
will now walk through. If you're impatient, you can just grab the [sty](/static/tex/poormanssubfig.sty)
file and jump ahead to the usage example.

We begin by declaring the package.

```latex
\NeedsTeXFormat{LaTeX2e}
\ProvidesPackage{poormanssubfig}[2025/02/14 A subfigure environment
  with working labels in the top-left corner that is compatible with revtex.]
```

Then, we create a new counter for the subfigures to be able to
generate use labels to reference them.

```latex
\newcounter{subfigure}
```

Now there comes the meat! The `multifig` environment resets the
subfigure counter[^fn:1], and redefines the `\thesubfigure` macro to
display references to the subfigure as `[Figure Number] ([figure
sublabel])`. The environment temporarily increases the Figure counter
so that `\thefigure` prints the right number.  The three commands at the
top of the below snipped allow the user to overwrite the way the label
is printed and formatted.

```latex
\newcommand{\subfiglabelformat}[1]{\raggedright{#1}}
\newcommand{\subfiglabelstyle}[1]{\alph{#1}}
\newcommand{\subfiglabelsize}{\small}
\newenvironment{multifig}%
{\addtocounter{figure}{1}%
  \setcounter{subfigure}{0}%
  \renewcommand\thesubfigure{\thefigure~(\subfiglabelstyle{subfigure})}%
}
{\addtocounter{figure}{-1}}
```

Now the hackery begins. We define an internal variables `\if@insubfig`
whose use will become clear later. The subfigure environment takes two
arguments, the first one being the width of the subfigure and the
second being an optional label string (it's also where you put
`\label{[label]}`). The environment then creates a minipage with the
appropriate width and prints the label of the figure in the top-left
corner.

```latex
\makeatletter
\ExplSyntaxOn
\newif\if@insubfig
\NewDocumentEnvironment{subfigure}{mO{}}{%
  \@insubfigtrue%
  \refstepcounter{subfigure}%
  \begin{minipage}{#1}%
    \subfiglabelformat{{\subfiglabelsize (\subfiglabelstyle{subfigure})~#2}}%
    }
    {\end{minipage}%
  \@insubfigfalse%
}
\ExplSyntaxOff
```

This final bit is truly horrendous. I wanted to be able to refer to
the subfigures in the figure caption using their labels, but a
standard `\ref{[label]}` would print the figure number, too. So I stole
some code from the [cleveref](https://ctan.org/tex-archive/macros/latex/contrib/cleveref) package to redefine the `\label` macro to
write a new label format (with the label name `[youlabel]@multifig`) to
the auxiliary (`.aux)` file upon compilation. LaTeX reads the auxiliary
file on the second pass and evaluates the `\newlabel{[label]}` macros
therein. These tell latex which text to print when we use the macro
`\ref{[label]}`. In our case this is just the sufigure counter value,
without the figure number. I define a new reference command `\subref` to expand `\ref` but
append `@multifig` to its argument, so that the text we stored using
`\newlabel` is used. These extra labels will only be generated for
subfigures thanks to `\if@insubfig`.

```latex
\let\multifig@old@label\label%
\def\label#1{%
  \multifig@old@label{#1}%
  \if@insubfig%
    \@bsphack%
    \protected@write\@auxout{}%
    {\string\newlabel{#1@multifig}{{(\subfiglabelstyle{subfigure})}%
        {\thepage}%
        {\@currentlabelname}%
        {\@currentHref}{}%
      }}%
    \@esphack%
  \fi
}%
\def\subref#1{\ref{#1@multifig}}
\makeatother
```

With these details out of the way, let's get to the part that you
actually wanted to read about[^fn:2]: The package is used in a similar way to -- who'd have thought
it -- the subfigure package, but puts your label it the top left
corner -- where it ought to be! Just put the `.sty` file somewhere `TeX`
will find it. For global installation, you can put it in your
`$TEXMFHOME/tex/latex` folder. For usage with a specific project, you
can dump it next to your `.tex` file. And now, behold the below usage example.

```latex
\begin{figure*}
  \begin{multifig}
    \begin{subfigure}{\columnwidth}{\label{subfig:descriptive-label}Some
        text label.}
      \begin{center}
        \includegraphics[width=.8\columnwidth]{example-image-a}
      \end{center}
    \end{subfigure}
    \begin{subfigure}{\columnwidth}{\label{subfig:another-descriptive-label}Another
        text label.}
      \begin{center}
        \includegraphics[width=.8\columnwidth]{example-image-b}
      \end{center}
    \end{subfigure}
  \end{multifig}
  \caption{\label{fig:beautiful-figures}As the discerning eye may have
    spied, the figures \subref{subfig:descriptive-label} and
    \subref{subfig:another-descriptive-label} have the potential to
    rock the foundations of society.}
\end{figure*}
Total nonsense, but I couldn't think of anything better to demonstrate
Figure \ref{subfig:descriptive-label} and \ref{subfig:another-descriptive-label}.

This produces the below result...
```

![](/ox-hugo/screen:b4ea3f2a-f6a3-467d-a9ae-56bb5f0b9084.png)
... and references in the text look like
![](/ox-hugo/screen:fef71aa0-4836-4777-8c42-1118c37c26b9.png)

Cheers!

[^fn:1]: Which could also have been accomplished by an
    argument to `\newcounter`.
[^fn:2]: Who are you dear reader? Shoot me an
    email!
