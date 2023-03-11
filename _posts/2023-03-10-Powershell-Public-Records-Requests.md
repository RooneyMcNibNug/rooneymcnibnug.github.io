---
layout: default
title:  "Power(shell) Pubic Records Requests"
categories: [Privacy]
tags: [foia, public records, powershell, transparancy]
---

![header_img](/img/ABF.png)

Over the past few years, I've done a lot of public records requests. I've also seperately done a lot of system administration with Powershell. What if...

# Power(shell) Pubic Records Requests

It dawned on me one day that knowing my way around [Powershell](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/00-introduction?view=powershell-7.3) might not just benefit my day job, but also help with creating a bit of transparency around government agencies.

It is no mystery that many public agencies are running their infrastructure with Microsoft products. This is especially true in terms of hosting email servers (Exchange is kind of a big thing still).

For the unfamilair to FOIA: doing requests against particular email addresses is a common practice. Once you know who you want emails from and have a timeframe, you can simply FOIA and see what the communications were like on that medium. "All emails from `user@domain.com` from date x to date y", for example.

Unfortunately, not all public agencies are very transparent about their email addresses. Its hard to know what emails to make a request for if you don't know what Mailboxes exist. Some would argue that publicizing this information would create a lapse in security and open their agencies up to spam. However, often the law says the public is entitled to this type of information being publicly disclosed via FOIA, and the agencies in question need to provide a compelling and lawful argument for excluding this type of information from such requests.

Requesting all staff email addresses is ultimately an option - its possible, but what else could we request in terms of emails? Maybe something that would glean more information about what the agency might be up to? What else could we get besides individual staff member email addresses that would be useful?

A switch was set off in my brain at some point, while working on some audits of mailboxes at work - "what kind of [Shared Mailboxes](https://learn.microsoft.com/en-us/microsoft-365/admin/email/about-shared-mailboxes?view=o365-worldwide) are these public agencies using?"

## Special Deliveries

Enter (sorta) artisinally-crafted Powershell one-liners within FOIA requests. For example:

> To Whom It May Concern:
>
> Pursuant to the Michigan Freedom of Information Act, I hereby request the following records:
>
> A CSV export of your agency's Shared Mailboxes (see https://learn.microsoft.com/en-us/microsoft-365/admin/email/about-shared-mailboxes?view=o365-worldwide for more info ) including the following parameters for every shared mailbox:
>
> - Primary SMTP Address
> - Date Created
> 
> A simple way to retrieve this is by having a member of IT (such as a System Administrator) run the following command in Exchange Online PowerShell (if email is hosted in a Microsoft Exchange environment):
> 
> `Get-Mailbox -ResultSize Unlimited -RecipientTypeDetails SharedMailbox | Select Name,PrimarySmtpAddress,WhenCreated | Export-CSV -Path .\SharedMailboxes_$((Get-Date).ToString('MM-dd-yyyy_hh-mm-ss')).csv`
> 
> The requested documents will be made available to the general public, and this request is not being made for commercial purposes.
>
> In the event that there are fees, I would be grateful if you would inform me of the total charges in advance of fulfilling my request. I would prefer the request filled electronically, by e-mail attachment if available or CD-ROM if not.
>
> Thank you in advance for your anticipated cooperation in this matter. I look forward to receiving your response to this request within 5 business days, as the statute requires.

Take note of the one-liner I included there:

```powershell
Get-Mailbox -ResultSize Unlimited -RecipientTypeDetails SharedMailbox | Select Name,PrimarySmtpAddress,WhenCreated | Export-CSV -Path .\SharedMailboxes_$((Get-Date).ToString('MM-dd-yyyy_hh-mm-ss')).csv
```

In digestable format, for those who have never touched powershell, this one liner is requesting the following:
- Get me all of your mailboxes on your email server, but only list the Shared Mailboxes (no individual user Mailboxes)
- For each Shared Mailbox found, give me the following metadata:
    * The Shared Mailbox name
    * The email address of the Shared Mailbox (in format `name@domain.com`)
    * The creation date of the Shared Mailbox
- Take all of this information and throw it into a CSV spreadsheet, where the spreadsheet will be titled with the date that this is one-liner is ran on

I did this for several agencies, and then waited..

## "You've Got Mail(boxes)!"

So what have my results looked like so far with these? A roundup so far:

- I have filed 69 of these duplicated requests to 69 different agencies
- At the time of writing, 17 of these requests have resulted in the agency in question running the one-liner (seemingly) and producing responsive documents
- 10 of the agencies have expressed that they "have no documents responsive to the request" (perhaps they don't use MSFT for their email infrastructure, or do not currently utilize any Shared Mailboxes)
- Only one agency haas asked for a fee - It was U.S. Customs and Border Protection. They wanted $984.00 to run the single line of code and go through the list of Shared Mailboxes produced..

I've been keeping the responsive documents [here on DocumentCloud](https://www.documentcloud.org/app?q=%2Bproject%3Ashared-mailboxes-210589%20) for now - I will be adding any future additions there as well.

So why even do this? Well, you can ascertain some interesting things from what Shared Mailboxes are being used by a public agency. Some of them have very specific names that match programs that the agency may be running, for example. Or emails dedicated to some sort of geography or business venture. What you can do once you have a list of these is find ones that pique your interest, and then from there you can file a seperate request(s) for email sent and/or received from that Shared Mailbox and see what you get.

Take for example, one of the emails I found in a responsive request was `CityBudgetData@seattle.gov`. Seems like a pretty good email to request data from with a FOIA if you are doing reporting on the government spending in that state!

## "I hope this email finds you well"

So what else can we do with one-liners in FOIA requests? Well, I have also been looking into enumerating [Alias Emails](https://learn.microsoft.com/en-us/microsoft-365/admin/email/add-another-email-alias-for-a-user?view=o365-worldwide) in the same way, with a simiatr template as the above, but the following seperate one-liner in powershell included:

```powershell
Get-Mailbox | Select-Object DisplayName,RecipientType,PrimarySmtpAddress, @{Name="Aliases";Expression={$_.EmailAddresses | Where-Object {$_ -clike "smtp:*"}}} | Export-Csv ./email_aliases.csv
```

[Here](https://www.muckrock.com/foi/chicago-169/foia-email-aliases-mayors-office-142375/) is an example of one of these.

I hope this encourages people to try and include some quick system commands within their FOIA requests. I was surprised at how many of these agencies responded to my requests the next day, producing a responsive .csv that was clearly created via the Powershell code I provided.

Of course, this type of excercise does not have to be limited to Powershell - I'm sure one could do a lot of interesting things with Bash or Perl one-liners.. heck, maybe I'll give that a go someday.

![header_img](/img/pwsh_git.png)
