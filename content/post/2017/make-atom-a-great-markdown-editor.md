---
title: Make Atom a great Markdown editor
date: '2017-09-07'
categories: [dev]
---

I use **Markdown** as much as I can but mainly to write blog posts and documentation. There are many great editors – at least on Mac – for Markdown but most of them are not free to use. Moreover, Ulysses just began a move, that others could follow, to a subscription model (~$5/month). Finally, I don’t want to avoid vendor lock-in and charging $40/year for plain text editing in a format designed to be usable from anywhere by **John Gruber** and **Aaron Swartz**.

> with the goal of enabling people “to write using an easy-to-read, easy-to-write plain text format, and optionally convert it to 
structurally valid XHTML (or HTML)”.[^1]

Among the Markdown editors my favorites are:

* iA Writer: A reference
* Byword: Another reference
* Caret: A new comer very promising
* Ulysses: The most powerful and also the less compliant with standards, that has moved recently to a subscription model
Most of them provide **the features I expect in any descent Markdown editor**.

**[Atom](https://atom.io/)** is a modern text editor that has every required features to be a killer app for professional looking Markdown editing.

Main pros

* It’s open source \o/
* Cross-platform
* Looks awesome :-)
* File view (tree)
* Multiple tabs
* Use the palette to open a file by it’s name (CMD+P)
* Autocompletion with tab
* Git support — obviously — if you want to track changes
* All you can expect from a powerful and modern text editor
* Plenty of customizations (UI + syntax) for all the tastes
* All you can imagine thanks to the plugin developed by the community

Here are the main packages / settings I use to **turn the Atom editor into an amazing Markdown editor**.

## Markdown Preview Plus

Markdown Preview Plus is an enhancement of the standard Markdown Preview that **renders Markdown documents** by applying a stylesheet.
I use it mainly for the following additional features

* Front matter (header / metadata) support
* Smart option to transform for example `--` into en dash (–)
* Math rendering (LaTeX) because I use it sometimes
For the last two features it’s necessary to enable Pandoc (it has to be installed) and to add the smart option[^2].

## Markdown-Writer

Mainly for **shortcuts** like cmd-b for **bold**. It’s necessary to create the default *keymap* to activate these shortcuts.

## UI & Syntax

I use **One** build-in UI (either in light or dark flavor) with **focus-light** or **focus-dark** syntax. I also like much the very original **flatwhite** syntax.

## Typewriter

To **center the text** on the screen instead of being left aligned.

## Keyboard-sounds

To add **keyboard sounds** while you are typing. I ♥ it.

## Wordcount

I don’t use it much but it’s a common feature of Markdown editors to display **file statistics**.
And the result is (with the *flatwhite* theme)

![atom markdown mode](/post/make-atom-a-great-markdown-editor_files/atom.png)

[^1]: [Markdown - Wikipedia](https://en.wikipedia.org/wiki/Markdown)
[^2]: [Pandoc User's Guide](http://pandoc.org/MANUAL.html)