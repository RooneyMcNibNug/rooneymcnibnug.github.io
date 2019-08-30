---
layout: default
title:  "A CentOS VPS with Wireguard and Pi-hole"
categories: [Privacy]
tags: [linux, privacy, pihole, wireguard]
---

# A CentOS VPS with Wireguard and PiHole

![header_img](/img/centos_wg_pihole.png)
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

## Finishing server configuration with new mobile client config

Now that we have things set up for the tunnel on our mobile config, we must now copy the public ("interface") key from your new there and paste it to ```wg0.conf``` on the server:

 ```console
[root@vps ~]# nano /etc/wireguard/wg0.conf

[Interface]
PrivateKey = !YOUR_PRIVATE_KEY!
ListenPort = 90210
SaveConfig = false
Address = 10.0.0.1/24

[Peer]
PublicKey = ~PUBLIC_KEY_ON_MOBILE~
AllowedIPs = 10.0.0.2/32
```

Looks nice - let's save the file and restart the wireguard service now with ```wg-quick up wg0```. You might want to set Wireguard up so that it starts up automatically with ```systemd``` after reboot. If this is what you want, let's do the following:

 ```console
[root@vps ~]# systemctl enable wg-quick@wg0
Created symlink from /etc/systemd/system/multi-user.target.wants/wg-quick@wg0.service to /usr/lib/systemd/system/wg-quick@.service.
```

## Traffic forwarding and routing configurations

Now we need to made sure that the traffic from Wireguard devices to this VM is re-routed to the proper interface. Make sure to run ```ip addr show``` to ensure that ```eth0``` is up and the proper name of the active interface connection here. Let's then pop open ```wg0.conf``` again and append the following ```iptables``` rules for proper traffic routing:

 ```console
[root@vps ~]# nano /etc/wireguard/wg0.conf

[Interface]
PrivateKey = !YOUR_PRIVATE_KEY!
ListenPort = 90210
SaveConfig = false
Address = 10.0.0.1/24
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

Let's also set ipv4 forwarding at kernel-level. We'll set this so that it remains persistent for the system even after reboot:

 ```console
[root@vps ~]# tee -a /etc/sysctl.d/99-sysctl.conf
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1
```

At this point you can either reboot the VM or run ```sysctl -p``` to apply these changes.

## Testing mobile-to-server connection

Finally, time to test from our phone. Let's open up the Wireguard app and toggle on the connection we created earlier. Once toggled, let's go back to our VM shell and check and run the following command to check the status of the ```wg0``` interface:

 ```console
[root@vps ~]# wg
interface: wg0
  public key: ~YOUR_SERVER_PUBLIC_KEY~
  private key: (hidden)
  listening port: 90210

peer: ~PUBLIC_KEY_ON_MOBILE~
  endpoint: ~IP~:~PORT~
  allowed ips: 10.0.0.2/32
  latest handshake: 3 seconds ago
  transfer: 1.43 KiB received, 92 B sent
  ```
  
Awesome, we can see that we have a handshake packets being received! As long as we see this going through, and we ensure that traffic is properly tunneling to the VM via Wireguard, we can now being to start setting up Pi-hole on the VM.
  
##  Preparing CentOS for Pi-hole installation

Before we go and install Pi-hole to our CentOS host, let's first take note of some of our network detaisl - particularly the IP for the ```wg0``` interface:

 ```console
[root@vps ~]# ip a show dev wg0
4: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.0.0.1/24 scope global wg0
       valid_lft forever preferred_lft forever
    inet6 ~REDACTED~/64 scope link flags 800
       valid_lft forever preferred_lft forever
```

Let's remember this ```inet scope``` output portion. In CentOS we also need to ensure that there is a proper network configuration file for the Wireguard interface created in the ```/etc/sysconfig/network-scripts/``` directory, in order for the Pi-hole installer to properly configure on this particular distro. Let's just create a blank network configuration in that directopry for the installer to write to like so:

```console
[root@vps ~]# touch /etc/sysconfig/network-scripts/ifcfg-wg0
```

Another slightly CentOS-specific thing we need to take into consideration is working with selinux, which comes installed by default on the OS. [According to Pi-hole developers](https://github.com/pi-hole/pi-hole/issues/752#issuecomment-513524149), selinux currently causes some issues with parts of Pi-hole. This means that having it in its default "enforcing" mode will cause issues here. This _does not_ mean that we should completely disable selinux - in fact, this is almost always an improper approach to issues with selinux.

Instead, let's set the mode on selinux here from "enforcing" to "permissive". This will not actively block things flagged up in your local selinux policy, but instead will still log any instances of policy violations:

```console
[root@vps ~]# cd /etc/sysconfig/
[root@vps ~]# cd nano selinux
    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    SELINUX=permissive
    # SELINUXTYPE= can take one of three values:
    #     targeted - Targeted processes are protected,
    #     minimum - Modification of targeted policy. Only selected processes are protected.
    #     mls - Multi Level Security protection.
    SELINUXTYPE=targeted
```

Let's save that file and then ```reboot``` in order to allow the changes to go through. Once we've started back up we can check the selinux status:

```console
[root@vps ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31
```

Looks set now. I encourage people to take a look at these logs from time to time. The live in ```/var/log/audit/audit.log``` on CentOS, so you can simply run ```cat /var/log/audit/audit.log | grep selinux``` at your desire.

During the Pi-hole install, we will also need to provide ```wg0``` as an interface name including your default gateway IP address such as ```192.168.x.x``` - this will be different on every server, so let's get our's now by checking ```ip r | grep default``` and keep the output handy. After saving these details, its time to actually start installing Pi-hole.

## Installing Pi-hole

If you don't have ```wget``` yet, please install it now as we will need it for getting the installer:

```console
[root@vps ~]# wget -O basic-install.sh https://install.pi-hole.net
[root@vps ~]# bash basic-install.sh
```

Let's follow these steps through the installation process:

* Allow basic-install.sh to install php on your system
* Aince we are using CentOS where SELinux is present (even if not enforced), we may not be able to use the web admin. Again, we should continue the installation without this, even though we put SElinux in "permissive" mode. Pi-hole admin via the CLI is [pretty straight-forward](https://docs.pi-hole.net/core/pihole-command/).
* Choose the ```wg0``` interface for pihole
* Select a DNS provider/server
* Select both ipv4 and ipv6 protocols
* Setup a static address (select "no" to default config to do this manually!)
* Change static ipv4 in next section to the wireguard server IP - ```10.8.0.1/24```
* Change gateway to your droplet's (```ip r | grep default``` from earlier from earlier)
* Accept settings after double checking
* Let's choose to not install the web admin interface, since we can check and change things via ```ssh``` on this VM if we need to
* We can also opt out of installing lighttpd for now if we'd like to avoid other issues
* Installation should be underway at this point

When the installation is done, we're just about set! Let's make sure that things look normal by checking DNS results from a known advertising domain that is on one of the default blocklists after Pi-hole install:

```console
[root@vps ~]# host track.adtrue.com 10.0.0.1
Using domain server:
Name: 10.0.0.1
Address: 10.0.0.1#53
Aliases: 

track.adtrue.com has address 0.0.0.0
track.adtrue.com has IPv6 address ::
track.adtrue.com is an alias for adtrue-track-server-1082517350.us-west-2.elb.amazonaws.com.
```

Looks like we're blocking it based on the 0'd address there. Let's also check out pihole log from the VM while we browse from our mobile, so that we can see that things are coming through properly and also being affectively blocked:

```console
[root@vps ~]# pihole -t
  [i] Press Ctrl-C to exit
21:10:34 dnsmasq[]: query[A] settings.crashlytics.com from 10.0.0.2
21:10:34 dnsmasq[]: /etc/pihole/gravity.list settings.crashlytics.com is 0.0.0.0
21:10:35 dnsmasq[]: query[A] analytics.twitter.com from 10.0.0.2
21:10:35 dnsmasq[]: /etc/pihole/gravity.list analytics.twitter.com is 0.0.0.0
21:10:35 dnsmasq[]: query[A] e.crashlytics.com from 10.0.0.2
21:10:35 dnsmasq[]: /etc/pihole/gravity.list e.crashlytics.com is 0.0.0.0
21:10:37 dnsmasq[]: query[A] reports.crashlytics.com from 10.0.0.2
21:10:37 dnsmasq[]: /etc/pihole/gravity.list reports.crashlytics.com is 0.0.0.0
21:10:51 dnsmasq[]: query[A] e.crashlytics.com from 10.0.0.2
21:10:51 dnsmasq[]: /etc/pihole/gravity.list e.crashlytics.com is 0.0.0.0
21:10:58 dnsmasq[]: query[A] reports.crashlytics.com from 10.0.0.2
21:10:58 dnsmasq[]: /etc/pihole/gravity.list reports.crashlytics.com is 0.0.0.0
21:11:06 dnsmasq[]: query[A] e.crashlytics.com from 10.0.0.2
21:11:06 dnsmasq[]: /etc/pihole/gravity.list e.crashlytics.com is 0.0.0.0
21:11:20 dnsmasq[]: query[A] reports.crashlytics.com from 10.0.0.2
21:11:20 dnsmasq[]: /etc/pihole/gravity.list reports.crashlytics.com is 0.0.0.0
21:11:33 dnsmasq[]: query[A] e.crashlytics.com from 10.0.0.2
21:11:33 dnsmasq[]: /etc/pihole/gravity.list e.crashlytics.com is 0.0.0.0
21:11:35 dnsmasq[]: query[A] connectivitycheck.gstatic.com from 10.0.0.2
^C
```
This output has been slightly edited here for privacy's sake, but you get the point. You might be surprised how often some of your mobile apps phone to ad domains (or you might not be surprised..)

We can check the dashbaord for more high-level infomration with the ```pihole -c``` command on the VM.

## Adding Pi-hole blocklists

At this point you can feel free to add any community-built ad/tracking blocklists to your Pi-hole config. The easiest way to do this from the VM is by adding each URL as a line to the ```/etc/pihole/adlists.list``` file and then updating Gravity with ```pihole -g```. 

Feel free to add [my list](https://raw.githubusercontent.com/RooneyMcNibNug/pihole-stuff/master/SNAFU.txt) if you'd like.

## Final considerations

So now you have your own personal VPS to tunnel into on the road, with DNS-level ad and tracker blocking as well. Neat! However, as quick painless as this has been, please do take the following into consideration as you continue to use this setup:

* Keep your CentOS system and underlying packages up to date by running ```yum upgrade``` when applicable
* Keep your Pi-hole services up to date by running ```pihole -up``` once and a while (you can check if there is an update to your service versions by running ```pihole -v``` first)
* Check you Android or iOS Wireguard app for updates
* You can always turn wireguard off server-side if you need to with ```wg-quick down```
* Take a look at ```netstat -tupln``` on CentOS to see if network settings are to your liking.

