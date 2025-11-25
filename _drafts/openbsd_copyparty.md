---
layout: default
title:  "Copyparty on OpenBSD - Sharing and Caring"
categories: [OpenBSD]
tags: [filesharing, openbsd, homelab]
---

# Copyparty on OpenBSD - Sharing and Caring


First, you'll need to download some dependencies, mostly python-related (`vim` here is just because I like that editor, its optional):
```
pkg_add -i ffmpeg py3-pip wget vim
pkg_add py3-aiohttp py3-humanize
```

Now donwload the [latest `.py` release](https://github.com/9001/copyparty/releases) file from Copyparty:
```
wget https://github.com/9001/copyparty/releases/latest/download/copyparty-sfx.py
```


```
touch partyhard.conf # configure things here to your liking, adding externally mounted drive
```

```
mkdir /mnt/usb
mount -o rw,noauto /dev/sdx/ /mnt/usb
```

```
python3 copyparty-sfx.py -c partyhard.conf
```

```
pkg_add ntfs_3g
doas ntfs-3g -o rw,noauto /dev/sd1i /mnt/usb
```

https://github.com/9001/copyparty/blob/hovudstraum/contrib/rc/copyparty
