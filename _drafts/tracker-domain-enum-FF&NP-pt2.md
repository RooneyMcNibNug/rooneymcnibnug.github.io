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


```console
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
