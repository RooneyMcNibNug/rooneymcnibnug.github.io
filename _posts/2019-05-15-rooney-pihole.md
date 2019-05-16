---
layout: default
title:  "Pi-hole and Tracker Domain Enumeration (For Fun and _Not_ Profit) - Part I"
categories: [privacy]
tags: [pihole, privacy]
---

# Pi-hole and Tracker Domain Enumeration (For Fun and _Not_ Profit) - Part I

## Blocking Ads & Trackers at DNS level with PiHole

It's a great idea to use ad-blocking at browser level with addons like [uBlock Origin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/) and [Privacy Possum](https://addons.mozilla.org/en-US/firefox/addon/privacy-possum/), but for casting a wider net around general marketing trackers on your network you should consider something closer to the router. This is where [Pi-hole](https://pi-hole.net/) comes in.

Pi-hole is a free and easy way to block ads and trackers at the DNS-level through-out your entire network. This not only makes it more probable that you'll be blocking trackers missed by your browser blockers, but also protects your other devices where the installation of one is not/less feasible. Perhaps a Google Home or a "smart" TV that won't shut up.

Pi-hole runs great on most [Raspberry Pi](https://www.raspberrypi.org/) models, but can otherwise be isntalled on practically any other *NIX device with the following command:

```console
curl -sSL https://install.pi-hole.net | bash
```
I also _highly_ recommend installing Pi-hole on a Virtual Private Server for mobile use. Ad and tracker blocking "on-the-go" is [absolutely possible](https://ifelse.io/2019/01/12/secure-ad-free-internet-anywhere-with-streisand-and-pi-hole/) by installing Pi-hole on a Linode, AWS, or other provider server alongside the proper VPN host configuration (you can whip this type of environment up quickly with something like [Streisand](https://github.com/StreisandEffect/streisand)). There are decent tutorials out there, but regardless of if you are using Android or iOS I can't say enough good things about the straight-forward configuration process and smooth connectivity of [WireGuard](https://www.wireguard.com/) as a VPN tunnel application.

A bit of a testament to the previous note about blocking "missed" trackers with this mobile VPN configuration - after having traffic running through a VPN connection such as the one above, you begin to see how much in-app tracking and ad queries you begin to catch:

![one fourth of this traffic is garbage..](/img/pihole_mobile.PNG)

Regardless of the platform you choose to install Pi-hole on, once you've got the install configured on the host as well as configured as your DNS resolver, you're good to start [adding blocklists](https://discourse.pi-hole.net/t/how-do-i-add-additional-block-lists-to-pi-hole/259).

However, being a huge fan of the project, I want to take this whole setup a step further and create my own blocklist to share with other Pi-hole users. After all, there's no short supply of domains to block.

## Popular Blocklists and their Coverage

A great collection of Pi-hole blocklists can be found at the [FilterLists](https://filterlists.com/) search page (you can filter for "Pi-hole" in the software field). Perhaps some of the most popular blocklists used by those running Pi-hole can be found [Firebog's Blocklist Collection page](https://firebog.net/), where they are grouped by type. My mission here is to run with these lists and inspect ads/tracking domains that are being "missed" by many of the blocklists included here and adding them to a new list to use and share.

Here is the output of my current blocklist config file for my Pi-hole:

```console
admin@pihole:~ $ cat /etc/pihole/adlists.list
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://mirror1.malwaredomains.com/files/justdomains
http://sysctl.org/cameleon/hosts
https://zeustracker.abuse.ch/blocklist.php?download=domainblocklist
https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt
https://hosts-file.net/ad_servers.txt
https://raw.githubusercontent.com/quidsup/notrack/master/trackers.txt
https://v.firebog.net/hosts/Easyprivacy.txt
https://v.firebog.net/hosts/Airelle-trc.txt
https://v.firebog.net/hosts/Shalla-mal.txt
https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV.txt
https://raw.githubusercontent.com/piwik/referrer-spam-blacklist/master/spammers.txt
https://raw.githubusercontent.com/hoshsadiq/adblock-nocoin-list/master/hosts.txt
https://www.squidblacklist.org/downloads/dg-ads.acl
https://v.firebog.net/hosts/Prigent-Ads.txt
https://raw.githubusercontent.com/anudeepND/blacklist/master/adservers.txt
https://v.firebog.net/hosts/static/w3kbl.txt
https://hosts-file.net/emd.txt
https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-malware.txt
https://bitbucket.org/ethanr/dns-blacklists/raw/8575c9f96e5b4a1308f2f12394abd86d0927a4a0/bad_lists/Mandiant_APT1_Report_Appendix_D.txt
https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt
https://v.firebog.net/hosts/Prigent-Malware.txt
https://v.firebog.net/hosts/Airelle-hrsk.txt
https://www.dshield.org/feeds/suspiciousdomains_High.txt
https://v.firebog.net/hosts/Easylist.txt
https://adaway.org/hosts.txt
https://hostsfile.org/Downloads/hosts.txt
https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-blocklist.txt
https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/android-tracking.txt
https://mirror.cedia.org.ec/malwaredomains/immortal_domains.txt
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/add.2o7Net/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/UncheckyAds/hosts
https://hosts-file.net/psh.txt
https://hphosts.gt500.org/hosts.txt
https://ransomwaretracker.abuse.ch/downloads/RW_DOMBL.txt
https://raw.githubusercontent.com/CHEF-KOCH/Canvas-Font-Fingerprinting-pages/master/Canvas.txt
https://raw.githubusercontent.com/CHEF-KOCH/WebRTC-tracking/master/WebRTC.txt
https://raw.githubusercontent.com/CHEF-KOCH/Audio-fingerprint-pages/master/AudioFp.txt
https://raw.githubusercontent.com/CHEF-KOCH/Canvas-fingerprinting-pages/master/Canvas.txt
https://zerodot1.gitlab.io/CoinBlockerLists/hosts
https://raw.githubusercontent.com/anudeepND/blacklist/master/facebook.txt
https://raw.githubusercontent.com/jerryn70/GoodbyeAds/master/Hosts/GoodbyeAds.txt
```

This puts me at almost 1 million blocked domains. After running with these blocklists for just under a year on one of the networks I use Pi-hole in, I have blocked 9.4% of the queries made within. This is with almost all devices being used within this particular network using browser-level adblocking as well. The Pi-hole runs seamless and I have close to zero issues with network traffic, minus the occasional need to whitelist a domain in order to get a site to function properly (this is easily done within the Pi-hole GUI or via the host terminal with ```pihole -w <domain>```). Again, it's re-assuring to have devices you wouldn't be able to control locally being watched over for tracking with Pi-hole.

So now I have the itch to dig into what types of domains are being blocked here, as well as what the current blocklist collection configured could be missing out on.

## Logging Queries

Viewing your "live" queries from the Pi-hole itself is made easy by using the command ```pihole -t```. Having this up while browsing is a nice way of inspecting what queries are being made when and how the Pi-hole config is affectively blocking domains.

Another option is having running this in the background and output the results to a file once you decide to terminate the command:

```console
admin@pihole:~ $ pihole -t > tail_queries.txt
```

We will come back to this file later.

If you want to see queries locally (on your laptop, etc) only, I recommend just using something like [tshark](https://www.wireshark.org/docs/man-pages/tshark.html) to filter for DNS traffic like so:

```console
local@machine:~ $ tshark -i wlan0 -f "src port 53" -n -T fields -e dns.qry.name
```

Again, piping the ```tshark``` output to a ```.pcap``` or ```.txt``` is a good idea here.

## Inspecting Queries and Building Your Own Blocklist

After letting the queries collect for enough time (I recommend even keeping the tail log running on the Pi-hole through a couple of days or so worth of traffic) we should start to inspect the output file.

While we could go through the ```.txt``` mnaually to look for queries that might be used for ads and/or tracking, let's parse it for relevant strings with ```egrep``` instead:

```console
admin@pihole:~ $ egrep -r 'track|ad|collect|stat|discover|metric|social|counter|lead|traffic|click|event|script|analytic|campaign|visit|engage|intel|monitor|audience|user|target|js|pulse|feed|tag|market|record|pixel|canvas|report' tail_queries.txt 
```
You can do a similar parse if you choose to use ```tshark```, of course.

These terms should be enough to bring to light any encroaching domains that are blocklists have missed. Anything that looks suspicious that _isn't_ logged like so:

```
...
22:03:25 dnsmasq[593]: /etc/pihole/gravity.list c.bing.com is 0.0.0.0
...
```

..is probably worth looking into. A simple search engine query of the domain in question can reveal a lot about the type of activity involved.

Anecdotally, some of the rabbit holes these not-yet-blocked domains have led me to some pretty bleak "marketing" imagery:

![you can't have your pudding if you don't eat your meat](/img/late_stage_marketing.jpg)

At this point, we have a bit of a decision to make in terms of how we want to be collecting these new domains to be blocked. Maybe per each root domain, we can choose to take one of the following actions:

1. Only add the exact domains/subdomains found from our last ```egrep``` outputs
2. Enumerate as much of each suspicious root domain as possible and pick outcome of subdomains to add to a blocklist from there

The first one could probably cause some issues to the functionality of certain sites. For example, one thing I did notice often is when I enumerated the root domain of some of these URLs I would often get a lot of internal or dev servers that the company may use, or some CDN domains that a site may use in order to serve up actually meaningful content besides ads. Considering this, Option 1 is probably safer.

If you do decide to go with Option 2, you will need to decide how you want to go about enumeration. In a pinch, you could use your search engine of choice to enumerate by running a search query of ```site:domain.com```. A better option would be using a tool like [Sublist3r](https://github.com/aboul3la/Sublist3r).

Sublist3r is a great tool for automating enumaration of a domain or subdomain using Python. Here is a usage example:

```console
local@machine:~ $ cd Sublist3r/
local@machine:~/Sublist3r $ ./sublist3r.py -d redshell.io

                 ____        _     _ _     _   _____
                / ___| _   _| |__ | (_)___| |_|___ / _ __
                \___ \| | | | '_ \| | / __| __| |_ \| '__|
                 ___) | |_| | |_) | | \__ \ |_ ___) | |
                |____/ \__,_|_.__/|_|_|___/\__|____/|_|

                # Coded By Ahmed Aboul-Ela - @aboul3la
    
[-] Enumerating subdomains now for redshell.io
[-] Searching now in Baidu..
[-] Searching now in Yahoo..
[-] Searching now in Google..
[-] Searching now in Bing..
[-] Searching now in Ask..
[-] Searching now in Netcraft..
[-] Searching now in DNSdumpster..
[-] Searching now in Virustotal..
[-] Searching now in ThreatCrowd..
[-] Searching now in SSL Certificates..
[-] Searching now in PassiveDNS..
[-] Total Unique Subdomains Found: 21
www.redshell.io
api.redshell.io
staging.api.redshell.io
application-api.redshell.io
beta.redshell.io
blog.redshell.io
www.blog.redshell.io
docs.redshell.io
prod-1.redshell.io
prod-2.redshell.io
prod-lb.redshell.io
staging.redshell.io
staging-api.redshell.io
staging-application-api.redshell.io
staging-beta.redshell.io
staging-mssql-application-api.redshell.io
staging-mssql-tracking-api.redshell.io
staging-tracking-api.redshell.io
t.redshell.io
www.t.redshell.io
tracking-api.redshell.io
```

You can also use the output option with ```-o``` to place throw these into a file if you want to.

I've been using Sublist3r for a good time now in this matter, and it has sometimes produced some interesting results of companies trying to spread their tracking through a smattering of similarly named subdomains:

![maybe they think this is cute or something](/img/track-list-seg.png)

## Checking it Twice

At this point we want to make sure we aren't already blocking some of the subdomains we've enumarated already on the Pihole-blocklists we already have confiugured. We can do this by searching for the root domain on the "Dashboard" GUI, or we can use ```grep``` again against the file where all our current blocklist domains are combined on the pihole itself:

```console
admin@pihole:~ $ grep -r "redshell.io" /etc/pihole/gravity.list 
api.redshell.io
redshell.io
admin@pihole:~ $ 
```

With the blocklist I'm making, I'm attempting to only add domains that aren't already added within the configured blocklists I currently have in my adlist configuration.

## The SNAFU Blocklist - A Work In Progress 

After going through all these steps, I've started compiling my very own blocklist, which I have titled "[SNAFU](https://en.wikipedia.org/wiki/SNAFU)". I'm currently hosting my list within a github repo - the raw file to add to your Pi-hole's ```adlists.list``` file can be found [here](https://raw.githubusercontent.com/RooneyMcNibNug/pihole-stuff/master/SNAFU.txt), and you can also search for it on filterlists.com, as it was added there recently.

For this list I have been using a combination of the methods mentioned here. This is obviously a work in progress. In my next post - Part II - I plan on attempting to combine a lot of these steps in order to automate the finding of "new" subdomains to block and adding them to my list. 
