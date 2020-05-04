---
layout: default
title:  "A Reasonable Network Printer Configuration in QubesOS"
categories: [Privacy, Secrurity]
tags: [linux, privacy, security, printing]
---

# A Reasonable Network Printer Configuration in QubesOS

Normally I try to avoid using a printer, but working with Public Records Requests and other document-heavy
projects makes something to scan and make hard copies with more of a necessity. 

I'm not fond of managing printers on _any_ system, so when I felt the need to do so on QubesOS it felt like something worth
documenting for posterity.

I followed some of the advice featured in the [Qubes Docs link pertaining to a network printer setup](https://www.qubes-os.org/doc/network-printer/)
but wanted to tweak some things to my own liking.

## Setting up a TemplateVM

I followed Qubes' advice in creating a TemplateVM for the printer drivers,
which I based on Fedora. I achieved this with a quick clone from `dom0`:

```console
[user@dom0 ~]$ qvm-clone fedora-30 printer-template
```

For a base, I wanted something simple. This TemplateVM doesn't need  packages like `thunderbird` and `gimp`, since we are only going to use it for minimal document management. This required a cursory look at `dnf list installed` to remove whatever packages I deemed unneccessary.

I wanted to make sure to add some tools imperative to printing and generally manageing different document types, so I added these particular packages to the TemplateVM:

```console
[user@printer-template ~]$ sudo dnf install system-config-printer qubes-pdf-converter qubes-img-converter iputils
```

## Setting up CUPS

After getting the TmplateVM settled, I started the Common UNIX Printing System configuration process:


```console
[user@printer-template ~]$ system-config-printer
```
- Download printer drivers in seperate VM
- Move to printer vm
- load manually into CUPs config

* this would all just be better to do on a standaloneVM at this point..
