---
layout: default
title:  "A CentOS VPS with Wireguard and Pi-hole"
categories: [Privacy]
tags: [linux, privacy, pihole, wireguard]
---

# A CentOS VPS with Wireguard and PiHole

I've evangilized how great I think both [Pi-hole](https://pi-hole.net/) and [Wireguard](https://www.wireguard.com/) are [in a previous post of mine](https://rooneymcnibnug.github.io/privacy/2019/05/15/rooney-pihole.html), where I explained how useful a combination of the two can be as a somehwat artisinal Virtual Private Server. I personally like this setup because it gives you full server access/permissions to a VPS service (at a low cost), something impossible to get with most managed VPN providers out there, while also an extremely easy and agreeable client setup.

## Getting CentOS 7 set up as VPS host

Why have I decided to go with CentOS as the host for this? The main factor here is stability. I want a system that I'm not going to have to check for package updates every day, and that I can trust to not break when I _do_ need to upgrade something. I don't need "bleeding edge".

I'm also personally very adjusted to doing admin on CentOS, like many other fulltime computer gazers out there. :stuck_out_tongue_winking_eye:

Using Linode, DigitalOcean, etc (choose your provider) its pretty simple to get a CentOS VM set up. For our VPS here, you will most likely want to chose the cheapest plan, as we only need minimal resources. I am currently running a CentOS VM with ~30GB disk space, 1GB RAM, and ~1GBPS network.

Once you've got CentOS set up on a VM, remember first to run ```yum upgrade``` in order to make sure all packages included are updated to their latest version.

## Installing Wireguard on the VM and setting server config

The next step is to install Wireguard on CentOS. At the time of writing this, WG i not yet included in one of the mainline repositories for CentOS, so we will need to add the repo manually first:

```console
[root@vps ~]# curl -Lo /etc/yum.repos.d/wireguard.repo https://copr.fedorainfracloud.org/coprs/jdoss/wireguard/repo/epel-7/jdoss-wireguard-epel-7.repo
[root@vps ~]# yum install epel-release
[root@vps ~]# yum install wireguard-dkms wireguard-tools
```
Once the packages have finished downloading, we need to createa a directory for the Wireguard config files and generate some keys:

```console
[root@vps ~]# mkdir /etc/wireguard
[root@vps ~]# wg set wg0-server listen-port 90210 private-key <(wg genkey)
[root@vps ~]# cd /etc/wireguard
[root@vps ~]# umask 077
[root@vps ~]# wg genkey | tee server-privatekey | wg pubkey > server-publickey #generate server-side keys
```
Let's also create the Wireguard server configuration file and adjust it accordingly:

```console
[root@vps ~]# nano /etc/wireguard/wg0.conf

[Interface]
PrivateKey = !YOUR_SERVER_PRIVATE_KEY!
ListenPort = 90210
SaveConfig = false
Address = 10.0.0.1/24
```

## Testing the server configuration

Okay, we've got the server config set up to where we need it so far. Let's bring up the Wireguard ```wg0``` interface with the applications ["quick" tool](https://git.zx2c4.com/WireGuard/about/src/tools/man/wg-quick.8) now to see if things will load properly:


```console
[root@vps ~]# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/xxx
[#] ip -4 address add 10.0.0.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
```

Looks like its up! Let's check the output of ```wg``` now as well:

```console
[root@vps ~]# wg
interface: wg0
  public key: ~YOUR_SERVER_PUBLIC_KEY~
  private key: (hidden)
  listening port: 90210
  ```
  
If things looks different than this, we might need to make sure our Wireguard install didn't go haywire or that we didn't make any mistakes with the key generation process. If ```wg-quick up``` went without errors, we are likely good and can down the service for now:
  
  ```console
[root@vps ~]# wg-quick down wg0
[#] ip link delete dev wg0
```

## Installing and configuring for mobile clients

One of the things I abolsutely love about Wireguard is their mobile apps. No more finicking with OpenVPN configurations and shoddy mobile applications! The Wireguard development team has an created great apps for both [Android](https://play.google.com/store/apps/details?id=com.wireguard.android) and [iOS](https://apps.apple.com/us/app/wireguard/id1441195209) systems. The configuration process for a mobile phone as a client to our currently existing server should be relatively the same for both platforms, so let's go setting this tunnel config up step by step for posterity:

* Click the ```+``` button to create a new config
  + “Create from scratch”
  + Give it a name
  + Click ```GENERATE``` beside "Private key” as well as "Public key"
  + Fill in ```10.0.0.2/32``` for “Addresses”
  + Fill in whatever DNS server details you want for now in "DNS servers" for now
* Peer information:
  + Click ```ADD PEER```
  + Fill in the server-public-key
  + Fill in ```0.0.0.0/0``` for “Allowed IPs”, so that while 'on-the-road'
  + Fill in the IP or domain-name with port-number for “Endpoint” (```x:x:x:x:90210```)
