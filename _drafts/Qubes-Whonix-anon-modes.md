---
layout: default
title:  "Automated Qubes AppVMs based on Whonix Anonymity Modes"
categories: [Privacy]
tags: [linux, privacy, Qubes, Salt]
---

# Automated Qubes AppVMs based on Whonix Anonymity Modes

<p align="center">
  <img width="312" height="287" src="/img/Qubes_Salt_Anon.png">
</p>

I love QubesOS. I use it as a daily driver for a lot of things at this point, especially compartmentalized communications. Many people I speak to use Qubes as well, so I often pick their brains about use cases, threat models, and what bells and whistles they've set up on their system.

I was speaking with one person in particular about how they manage anonymization schemes with AppVMs, and they told me about how they rely on following guidelines put forth by Whonix in their wiki (located on the Whonix Anonymity Modes page). This idea makes a lot of sense to me, and I was eager to try it out.

### Understanding Anonimity Mode levels

To me, a fundamental consideration with these different anonymity modes is data retention. If you are posting something to a public forum with a username connected to your actual identity in one way or another, its good to get into the mindset that you left a piece of you somewhere. The internet in constantly mined for user metadata: your contact information, your "likes", your political opinions and ideologies, etc. It is every marketing agency's dream to be able to tie everyone's account details to one another to paint a mosiac of a user's desires - data brokers as intelligence agencies. There are already Open Source Intelligence tools that do just this.

Thus it is necessary to use threat modeling and carefully consider use cases based on activity against the proper tools and connections.


### Anonimity Mode use-cases


### Generating the AppVMs with Salt

