---
layout: default
title:  "Power(shell) Pubic Records Requests"
categories: [Privacy]
tags: [foia, public records, powershell, transparancy]
---

![header_img](/img/ABF.png)

Over the past few years, I've done a lot of public records requests. I've also seperately done a lot of system administration with Powershell. What if...

# Power(shell) Pubic Records Requests

It dawned on me one day that knowing my way around Powershell might not just benefit my day job, but also help with creating a bit of transparency around government agencies.

It is no mystery that many public agencies are running their infrastructure with Microsoft products. This is especially true for using Exchange in terms of hosting email servers.

For the unfamilair to FOIA: doing requests against particular email addresses is a common practice. Once you know who you want emails from and have a timeframe, you can simply FOIA and see what the communications were like on that medium.

Unfortunately, not all public agencies are very transparent what email addresses are being used. Some would argue doing so would create a lapse in security and open their agencies up to spam. However, often the law says the public is entitled to this type of information being publicly disclosed via FOIA, and the agencies in question need to provide a compelling and lawful argument for exclusing this.

Requesting all staff email addresses is one thing - its possible, but what else could we request in terms of emails? Maybe something that would glean more information about what the agency might be up to?

A switch was set off in my brain at some point, while working on some audits of mailboxes at work - "what kind of shared mailboxes are these public agencies using?"

## Special Deliveries

Enter artisinally-crafted Powershell one-liners within FOIA requests. 

## "You've Got Mail(boxes)!"
