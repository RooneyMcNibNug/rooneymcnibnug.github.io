---
layout: default
title:  "Copyparty on OpenBSD - Sharing and Caring"
categories: [OpenBSD]
tags: [filesharing, openbsd, homelab]
---

# Copyparty on OpenBSD - Sharing and Caring

I have a few OpenBSD boxes on the net and at home. One near me is running on a Pi 3, even - it works great, recently upgraded to [7.8](https://www.openbsd.org/78.html)!

Recently I had the need to whip up a quick share-out of files on an external drive to someone else on the network, so I remembered Copyparty [https://github.com/9001/copyparty] exists and figured I could use this Pi that isn't doing as much as other machines atm.

But could I get it to run on OpenBSD? Yeah, no problem. And you can two, ince case you have a similar set of needs or are just in a the mindset to give it a try:

First, you'll need to download some dependencies, mostly python-related (`vim` here is just because I like that editor, obviously use whatever editor you want):
```
pkg_add -i ffmpeg py3-pip wget vim
pkg_add py3-aiohttp py3-humanize
```

Now download the [latest `.py` release](https://github.com/9001/copyparty/releases) file from Copyparty:
```
wget https://github.com/9001/copyparty/releases/latest/download/copyparty-sfx.py
```

Also optional, but very useful - creating a configuration file for the server to use: 
```
touch partyhard.conf # configure things here to your liking, adding externally mounted drive
```

[Here](https://github.com/9001/copyparty/blob/hovudstraum/docs/example.conf) is a sample configuration to take a look at, with useful comments.

I am using pretty simple things here - anyone visiting has read access and there are credentials for admin seperate from that.

You'll also want to specify in that config what drive you want to share out - here I am using a usb which I will mount like so:

```
mkdir /mnt/usb
mount -o rw,noauto /dev/sdx/ /mnt/usb # replace /dev/sdx with your actual drive location
```

A NOTE: If you are using an `NTFS` drive, with OpenBSD in particular you need to do some different steps to get it mounted properly:

```
pkg_add ntfs_3g
doas ntfs-3g -o rw,noauto /dev/sdx /mnt/usb # replace /dev/sdx with your actual drive location
```

When you are all set with your config, you can start the service with `python`:

```
python3 copyparty-sfx.py -c partyhard.conf
```
And ther you go - the output of this last command should print some options with the local IP/hostname as well as port so that you can access the copyarpty server from various devices on the local network.

If you want to get you can tweak this `rc` script as well and add it to your host: 
https://github.com/9001/copyparty/blob/hovudstraum/contrib/rc/copyparty

Sure, there are plenty of other ways to host things with OpenBSD, but if you are looking to move fast and make soemthing available in a number of ways on the local network, this might work for you. It's done wonders for me in a few quick instances, anyways.
