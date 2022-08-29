---
layout: default
title:  "Pi-hole and Tracker Domain Enumeration (For Fun and _Not_ Profit) - Part II"
categories: [Privacy]
tags: [linux, privacy, pihole]
---

# Pi-hole and Tracker Domain Enumeration (For Fun and _Not_ Profit) - Part II

In an [old post](https://rooneymcnibnug.github.io/privacy/2019/05/15/rooney-pihole.html) on this blog I wrote about the joys of blocking creepy advertising and tracking domains, and how this strange little hobby sparked a desire for me to build [my own blocklist](https://raw.githubusercontent.com/RooneyMcNibNug/pihole-stuff/master/SNAFU.txt).

This blog post will serve as the ever-belated second part, where we can go over an interesting source example for subdomain enumeration for the `SNAFU` blocklist. So let's get right into it.

## Going to Market

I have used a lot of methods to build my blocklist - from manually glimpsing through queries and finding ones that look like they might need to be blocked, from more automated methods. I kept thinking it might be nice to document one of the automated ones, when I stumbled upon a great example.

One of the sources I use for find new ad/tracking domains to enumerate is something called [Martech](https://martech.org/what-is-martech/), who advertise themselves as the following:

> "MarTech’s mission is unlike any other. You’ll get the mainstream marketer’s perspective: innovative, practical, brand-safe, results-driven … and resource- and time-constrained. The quest is unearthing the universal challenges marketers face. The holy grail is revealing the countless solutions they devise to succeed in today’s customer-centric, digital-first, and multichannel marketing environment."
>
> "In short, MarTech is Marketing."

Martech has had some interesting visualizations on different "players" in the digital marketing space in the past, which they call the "Martech Marketing Landscape". They release one of these every year and I have been [collecting](https://github.com/RooneyMcNibNug/pihole-stuff/tree/master/martech_landscape_imgs) them in order to scope out new domains to enumerate in a search for new entries to add to my `SNAFU` blocklist.

![image](https://user-images.githubusercontent.com/17930955/178816597-332b6e91-7590-48dc-bc35-b7bfe171bc19.png)
That's quite a creepshow! Perhaps it would be worth the effort to go forth and cast the net out to reel in new invasive URLs?

This has been relatively easy in the past, but when I went to check out this year's "Landscape" I was a bit suprised to find a couple of details that were bound to make it harder to grab domains from:

1. You need to create an account to [access](https://martechmap.com/int_supergraphic) the graphic.
2. The graphic is dynamic and does not contain domains up front - you have to hover over icons for them to be displayed:

<img width="795" alt="image" src="https://user-images.githubusercontent.com/17930955/179120301-66ad66de-db67-4cc7-98f1-7957780abb55.png">

## Doing the legwork

So with a quick look at the [Inspector tool in Firefox](https://firefox-source-docs.mozilla.org/devtools-user/page_inspector/how_to/open_the_inspector/index.html), I was able to find the URL containing the source for the Landscape. From there, I could see the html source code:

<img width="1328" alt="image" src="https://user-images.githubusercontent.com/17930955/179120893-132e85b7-47e6-42ff-90fa-409761d046ab.png">

I figured it would be best to dump all of the _unique_ URLs into a text file, so I did so on my nearest linux distro with the following one-liner:

```console
boop@pihole:~ $ grep "<a href=" 2022_martech_source.txt | grep -Eo "(www.)[a-zA-Z0-9./?=_%:-]*" | sort -u > 22mardomains.txt
```

After this, a little clean-up was required. There was a bit of manual work in finding domains that were included on the page outside the scope of the Landscape itself (it was littered with other LinkedIn URLs, for example. Once I had done that, I wanted to omit the `www.` from every line in the file, which I did with the following:

```console
boop@pihole:~ $ sed 's/^....//' 22mardomains.txt > 22marfixed.txt
```

Now it was time to enumerate. In my previously mentioned post I referenced a tool called [Sublist3r](https://github.com/aboul3la/Sublist3r), which I used in the past for this part. I have been using [Findomain](https://github.com/Findomain/Findomain) recently as a replacement for a few reasons, but mostly because it includes enumeration through Sublist3r as well as many other tools combined and works better running with a file for input.

The plan was to run Findomain against my `22marfixed.txt` file , so that it could go and recursively run against every domain within and spit it all out into a new file I would need later:

```console
boop@pihole:~ $ findomain -f 22marfixed.txt -r -u 22MarEnums.txt
```

Here is an exmaple Findomain running aginst one of the domains in `22marfixed.txt`:

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

With >9,000 domains in my list `22marfixed.txt`, this part was the most time consuming. I left it running for a few days before it was able to finish full enumerations for all the domains provided.

When it was finally complete, I did also have to dig through and find domains that might break things for anyone using the `SNAFU` list before adding it to there. There were some other sections on the Martech page within the dynamic Landscape that were less applicable to user monitoring/tracking, ad-serving, etc. and more along the lines of point-of-sales systems, authenitcations, etc.

Once I had scathed through `22MarEnums.txt` and cleaned it up, I needed to dump all of the current domains being blocked on my pi-hole. I wanted to dump _not just_ the domains from the blockist, but instead _all_ lists I was currently using, which include other popular lists. The reason for this is I designed `SNAFU` to try and be a list that would be a great suppliment alongside other lists that people have worked hard on harvesting domains to block - that way I am only grabbing things that have been missed from some of the other lists who have been around longer.

In the past year or so, Pi-hole has switched to Sqlite3 for its [Gravity database](https://docs.pi-hole.net/database/gravity/), which includes all of the blocked domains. `gravity.db` was where I needed to look to see all of those. 

I went ahead figured out the SQL syntax to grab all of the domains and dump them into ~another~ seperate text file.

```console
boop@pihole:~ $ sudo sqlite3 /etc/pihole/gravity.db "SELECT domain FROM gravity WHERE rowid IN (SELECT rowid FROM gravity GROUP BY domain);" > /home/pi/GravDump.txt"
```

```console
boop@pihole:~ $ wc -l GravDump.txt
1296697 GravDump.txt
```

I compared this to the number of blocked domains showing up on the pi-hole admin GUI, and it checked out:

<img width="250" alt="image" src="https://user-images.githubusercontent.com/17930955/179126866-ec0f8eb9-1210-4bf1-9895-2fe794757965.png">

Okay, so now we were at the de-duplicating stage. We want to try and find any URLs we enumerated within `22MarEnums3.txt` and "de-duplicate" them from our `GravDump.txt`. A kind of janky way I went about this was using the `diff` command in the following way:

```console
boop@pihole:~ $ diff -U $(wc -l < 22MarEnums3.txt) 22MarEnums.txt GravDump.txt| sed -n 's/^-//p' > ToAdd.txt
```

## Building a bigger shield

Remember when I said the bulk `findomain` job was the part of all this that was most time-consuming? I lied.

The `ToAdd.txt` file was _massive_, clocking at >45k domains (each a line-item) within. It would be quicker to automatically process through this file, maybe using [pieces of another script](https://github.com/RooneyMcNibNug/pihole-stuff/blob/master/python_scripts/pihole_domain_finder.py#L4) for picking out relevant malicious domains to add to the pi-hole database and cutting through benign ones, but this would miss out on more "full" enumerations of entire domains that are dedicated to tracking/ads. I want to block as much from those shady sites as possible with my list.

Ultimately, this is requiring me to go through each one of these root domains and check out the sites a bit, to see what services they offer. If it is not inherently a site dedicated to providing services for advertising/tracking in any way, I can narrow it down to just keeping subdomains with addresses like `analytics.<domain_name>.com` or `pixel.<domain_name>.io` and so on. But when I find out that `yourclicksareourbusiness.xyz` is aptly named, I keep the entire domain enumeration (all subdomains) to block.

This results in me going alphabetically through the file and having to do some [rather large PRs](https://github.com/RooneyMcNibNug/pihole-stuff/commit/dc3338782332c5f668790f07f5216b8f65c2cdde) within my dedicated Github repo. I'm taking this piece-meal, but hoping to have a much more powerful and all-encompassing blocklist when I am finished, for all to use and share freely.

I'm grateful for anyone who dedicated a bit of time to make using the internet a safer and less painful experience, so this has become a bit of a labour of love for me. I'm sure there are ways I can better build stuff like this, and I am always open to conversation on the matter. I will say that if you want to support the `SNAFU` blocklist, the best thing that you can do is use it and [submit an issue](https://github.com/RooneyMcNibNug/pihole-stuff/issues) when you run into something like a false positive or a site you find important not working properly due to a subdomain I have included in the blocklist. I try to respond to these and fix them within 48hrs.

Until next time, be well.
