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

For the unfamilair to FOIA: doing requests against particular email addresses is a common practice. Once you know who you want emails from and have a timeframe, you can simply FOIA and see what the communications were like on that medium. "All emails from `user@domain.com` from date x to date y", for example.

Unfortunately, not all public agencies are very transparent what email addresses are being used. Its hard to know what emails to make a request for if you don't know what mailboxes exist! Some would argue doing so would create a lapse in security and open their agencies up to spam. However, often the law says the public is entitled to this type of information being publicly disclosed via FOIA, and the agencies in question need to provide a compelling and lawful argument for exclusing this.

Requesting all staff email addresses is one thing - its possible, but what else could we request in terms of emails? Maybe something that would glean more information about what the agency might be up to?

A switch was set off in my brain at some point, while working on some audits of mailboxes at work - "what kind of [shared mailboxes](https://learn.microsoft.com/en-us/microsoft-365/admin/email/about-shared-mailboxes?view=o365-worldwide) are these public agencies using?"

## Special Deliveries

Enter artisinally-crafted Powershell one-liners within FOIA requests. 

```powershell
Get-Mailbox -ResultSize Unlimited -RecipientTypeDetails SharedMailbox | Select Name,PrimarySmtpAddress,WhenCreated | Export-CSV -Path .\SharedMailboxes_$((Get-Date).ToString('MM-dd-yyyy_hh-mm-ss')).csv
```

In digestable format, this one liner is requesting the following:
- Get Me all of your mailboxes on your email server, but then only list the Shared Mailboxes (no individual user mailboxes)
- For each Shared Mailbox found, give me the following metadata:
    * The Shared Mailbox name
    * The email address of the Shared Mailbox (`name@domain.com`)
    * The creation date of the Shared Mailbox
- Take all of this information and throw it into a CSV spreadsheet, where the spreadsheet will be titled with the Date that this is ran on.


## "You've Got Mail(boxes)!"

So what have my results looked like so far with these? A roundup so far:

- I have filed 69 of theese duplicated requests to 69 different agencies
- At the time of writing, 17 of these requests have resulted in the agency in question running the one-liner and producing responsive documents
- 10 of the agencies have expressed that they "have no documents responsive to the request" (perhaps they don't use MSFT for their email infrastructure, or do not currently utilize any shared mailboxes)
- Only one agency haas asked for a fee - It was U.S. Customs and Border Protection. They wanted $984.00 to run the single line of code and go through the list of Shared Mailboxes produced..


I've been keeping the responsive documents here on DocumentCloud for now - I will be adding any that come back in the future there as well.

## I hope this finds you well

So what else can we do here? Well, I have also been looking into enumerating [Alias Emails](https://learn.microsoft.com/en-us/microsoft-365/admin/email/add-another-email-alias-for-a-user?view=o365-worldwide) in the same way, with a simiatr template as the above, but the following seperate one-liner in powershell included:


Here is an example of one of these.

I hope this encourages people to try and include some quick system commands within their FOIA requests. I was surprised at how many of these agencies responded to my requests the next day, producing a responsive .csv that was clearly created via the powershell code I provided.

Of course, this does not have to be limited to powershell - I'm sure one could do a lot of interesting things with bash or perl oneliners.. heck, maybe I'll give that a go someday.

![header_img](/img/pwsh_git.png)
