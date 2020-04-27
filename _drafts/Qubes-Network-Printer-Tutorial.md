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

## Getting Started

The printer I am using is a Cannon PIXMA Scanner/Printer model. I followed Qubes' advice in creating a TemplateVM for the printer drivers,
which I based on Fedora. I achieved this with a quick clone from `dom0`:

```console
[user@dom0 ~]$ qvm-clone fedora-30 printer-template
```

This TemplateVM can be stripped of a lot of non-essential packages like `thunderbird` and `gimp`, since we are only going to
use it for minimal document management. If you would rather, you could just get a fresh Fedora template generated to work from (or [reinstall](https://www.qubes-os.org/doc/reinstall-template/)) instead of cloning an existing one:

```console
[user@dom0 ~]$ sudo qubes-dom0-update qubes-template-fedora-30
```

I wanted to make sure this VM had DisposableVMs enabled, since I really don't trust printers:

```console
[user@dom0 ~]$ qvm-create --template printer-template --label red document-print-dvm
[user@dom0 ~]$ qvm-prefs document-print-dvm template_for_dispvms True
[user@dom0 ~]$ qvm-features document-print-dvm appmenus-dispvm 1
```
