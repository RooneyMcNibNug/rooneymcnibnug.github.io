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
  
##  Preparing for Pi-hole installation

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

---------------------------------------

# I think we have to change SELinux status to "permissive" (https://github.com/pi-hole/pi-hole/issues/752#issuecomment-513524149). We might want to do this permanently for the system:
$ cd /etc/sysconfig/
$ nano selinux
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
$ reboot
$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31

# You also need to provide wg0 as an interface name including your default gateway IP address such as 192.168.2.1 (this is different for every server, get & save your own by below command):
$ ip r | grep default

# Save these details and then start getting/running pihole install:
$ wget -O basic-install.sh https://install.pi-hole.net
$ bash basic-install.sh

    #> Allow basic-install.sh to install php on your system
    #> If you use Fedora or CentOS where SELinux is present and enforced, you may not be able to use the web admin. I'm okay        with this and continued the installation without fully disabling selinux.
    #> Choose the "wg0" interface for pihole
    #> Select DNS server
    #> Select both 1pv4 and ipv6 protocols
    #> Setup a static address (select "no" to default config to do this manually!)
    #> Change static ipv4 in next section to the wireguard server IP - 10.8.0.1/24
    #> Change gateway to your droplet's ("ip r | grep default" from earlier)
    #> Accept settings after double checking
    #> I chose to not install the web admin interface, since I can check and change things via ssh on this droplet
    #> I opted out of installing lighttpd for now..
    #> Installation should be underway

# Time 2 test
$ host eff.org 10.0.0.1

# Now start using your phone and check these for a while
$ pihole -t
$ pihole -c



