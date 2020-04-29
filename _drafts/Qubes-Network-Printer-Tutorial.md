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

For a base, I wanted something simple. This TemplateVM doesn't need  packages like `thunderbird` and `gimp`, since we are only going to use it for minimal document management. For this reason I chose building a [Minimal TemplateVM](https://www.qubes-os.org/doc/templates/minimal/) for this:

```console
[user@dom0 ~]$ sudo qubes-dom0-update qubes-template-fedora-30-minimal
```

I wanted to make sure this VM had DisposableVMs enabled, since I really don't trust printers:

```console
[user@dom0 ~]$ qvm-create --template printer-template --label red document-print-dvm
[user@dom0 ~]$ qvm-prefs document-print-dvm template_for_dispvms True
[user@dom0 ~]$ qvm-features document-print-dvm appmenus-dispvm 1
```

Since this is a Minimal TemplateVM, I wanted to make sure to add some tools imperative to printing and generally manageing different document types. I set up `qubes-core-agent-passwordless-root` first following [these instructions](https://www.qubes-os.org/doc/vm-sudo/). Afterwards I added these packages to the TemplateVM:

```console
[user@printer-template ~]$ sudo dnf install qubes-pdf-converter qubes-img-converter libreoffice vim envice
```
