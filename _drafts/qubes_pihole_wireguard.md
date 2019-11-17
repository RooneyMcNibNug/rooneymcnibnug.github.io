---
layout: default
title:  "Using a Pi-Hole VPS with QubesOS via Wireguard"
categories: [Privacy]
tags: [linux, privacy, security, pihole, wireguard, qubes]
---

# QubesOS VPN AppVM: Wireguard to Pi-hole VPS

## Here is an outline of what I want to accomplish here:

- As I see it, there are two options:
1. Virtualize a Pi-hole within a Qubes AppVM in order to route traffic from other VMs to it "on the go"
2. Set up a Wireguard connection from an AppVM in order to hit my external VPS already set up with the Pi-hole on it

- There is already a pretty good write-up on setting up an AppVM for a VPN connection, and a recent one at that: https://micahflee.com/2019/11/using-mullvad-in-qubes/

- This write-up handles setting up a Wireguard AppVM in debiab, but I'd prefer to tweak it to handle it in Fedora: https://github.com/tasket/Qubes-vpn-support/wiki/Wireguard-VPN-connections-in-Qubes-OS

- Doing it in Fedora should be easy, as Wireguard has a copr repo 
```console
[usr@appVM ~]$ sudo dnf copr enable jdoss/wireguard
```
