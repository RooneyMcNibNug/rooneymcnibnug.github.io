---
layout: default
title:  "Pi-hole and Tracker Domain Enumeration (For Fun and _Not_ Profit) - Part II"
categories: [Privacy]
tags: [linux, privacy, pihole]
---

GIST:

```
- https://chiefmartec.com/2022/05/marketing-technology-landscape-2022-search-9932-solutions-on-martechmap-com/
- register with throwaway email to access https://martechmap.com/int_supergraphic
- inspect element and point to the proper file
- save source as "2022_martech_source.txt"
- dump all unique hyperlinks from the source: $ grep "<a href=" 2022_martech_source.txt | grep -Eo "(www.)[a-zA-Z0-9./?=_%:-]*" | sort -u > 22mardomains.txt
- get rid of 'www.' on each line to easier digest: $ sed 's/^....//' 22mardomains.txt > 22marfixed.txt
- enaumerate the unique domains to get all sub-domains: $ findomain -f 22marfixed.txt -r -u 22MarEnums.txt
- might have to manually remove domains that might break things..
- grep the enumerations file for common terms: $ egrep -r "track|ad|collect|stat|discover|metric|social|counter|lead|traffic|click|event|script|analytic|campaign|visit|engage|intel|monitor|audience|user|target|js|pulse|feed|tag|market|record|pixel|canvas|report" 22MarEnums.txt
- dump domains currently stored in gravity.db: $ sudo sqlite3 /etc/pihole/gravity.db "SELECT domain FROM gravity WHERE rowid IN (SELECT rowid FROM gravity GROUP BY domain);" > /home/pi/GravDump.txt"
```

WRITE UP:

In an [old post](https://rooneymcnibnug.github.io/privacy/2019/05/15/rooney-pihole.html) on this blog I wrote about the joys of blocking creepy advertising and tracking domains, and how this joy sparked a desire for me to build [my own blocklist](https://raw.githubusercontent.com/RooneyMcNibNug/pihole-stuff/master/SNAFU.txt).

This blog post will serve as the ever-belated second part, where we can go over an interesting source example for sub-domain enumeration for the `SNAFU` blocklist. So let's get right into it.

## Going to Market

I have used a lot of methods to build my SNAFU blocklist - from manually glimpsing through queries and finding ones that look like they might need to be blocked, from more automated methods. I kept thinking it might be nice to document one of the automated ones, when I stumbled upon a great example.

One of the sources I use for find new ad/tracking domains to enusmerate is soemthign called [Martech](https://martech.org/what-is-martech/), who advertise themselves as the following:

> MarTech’s mission is unlike any other. You’ll get the mainstream marketer’s perspective: innovative, practical, brand-safe, results-driven … and resource- and time-constrained. The quest is unearthing the universal challenges marketers face. The holy grail is revealing the countless solutions they devise to succeed in today’s customer-centric, digital-first, and multichannel marketing environment. 
>
> In short, MarTech is Marketing. 

Martech has had some interesting visualizations on different players in the digital marketing space in the past, which they call the "Martech Marketing Landscape". They release one of these every year and I have been [collecting](https://github.com/RooneyMcNibNug/pihole-stuff/tree/master/martech_landscape_imgs) them in order to scope out new domains to enumerate in a search for new entries to add to my SNAFU blocklist.


![image](https://user-images.githubusercontent.com/17930955/178816597-332b6e91-7590-48dc-bc35-b7bfe171bc19.png)
That's a lot of crap! Perhaps someone should 

```consoleof a 
boop@pihole:~ $ wc -l GravDump.txt
1296697 GravDump.txt
```

Findomain enum:

```
Running wildcards detection for 100shoppers.com...
No wilcards detected for 100shoppers.com, nice!

Performing asynchronous resolution for 17 subdomains for the target 100shoppers.com, it will take a while...

widget-lv.100shoppers.com
widget-lt.100shoppers.com
shop-at.100shoppers.com
app.100shoppers.com
widget-ee.100shoppers.com
100shoppers.com
widget-at.100shoppers.com
files-lv.100shoppers.com
widget.100shoppers.com
shop-lt.100shoppers.com
docs.100shoppers.com
files-ee.100shoppers.com
help.100shoppers.com
files-at.100shoppers.com
www.100shoppers.com
files-lt.100shoppers.com
shop-ee.100shoppers.com

Job finished in 11 seconds.
>> 📁 Subdomains for 100shoppers.com were saved in: ./22MarEnums.txt 😀

Good luck Hax0r 💀!

Rate limit set to 5 seconds, waiting to start next enumeration.
```
