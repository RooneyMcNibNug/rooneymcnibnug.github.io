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

- This write-up handles setting up a Wireguard AppVM in debian, but I'd prefer to tweak it to handle it in Fedora: https://github.com/tasket/Qubes-vpn-support/wiki/Wireguard-VPN-connections-in-Qubes-OS

- Doing it in Fedora should be easy, as Wireguard has a copr repo 

![AppVM creation](/img/pihole-appvm.png)

```console
[user@fedora-30 ~]$ sudo dnf copr enable jdoss/wireguard
[user@fedora-30 ~]$ sudo dnf install wireguard-dkms wireguard-tools
```

```console
[user@pihole-VPN ~]$ sudo mkdir /etc/wireguard
[user@pihole-VPN ~]$ cd /etc/wireguard
[user@pihole-VPN ~]$ wg genkey | tee clientprv | wg pubkey > clientpub
[user@pihole-VPN ~]$ nano /etc/wireguard/wg0.conf
```
generate the keys , yada yada

^ really can't all of the above just be 'refer to previous post' here? https://rooneymcnibnug.github.io/privacy/2019/08/30/CentOS7-Wireguard-PiHole.html

- make sure to share use cases for this any why it is import to decide carefully which VMs to link to this network option.

- build the client wg configuration in the root AppVM directory

- remember to add your AppVM public key and allow it through wireguard on the erver via ```wg set wg0 peer <client-public-key> allowed-ips 10.10.0.2/32```

- official Qubes VPN documentation: https://www.qubes-os.org/doc/vpn/
