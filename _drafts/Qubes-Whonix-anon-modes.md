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

To me, a fundamental consideration with these different anonymity modes is data retention. If you are posting something to a public forum with a username connected to your actual identity in one way or another, its good to get into the mindset that you "left a piece of you" somewhere. The internet in constantly mined for user content and  metadata: your contact information, your "likes", your political opinions and ideologies, etc. It is every marketing agency's dream to be able to tie everyone's account details to one another to paint a mosiac of a user's desires - data brokers have big dreams of operating as if they were intelligence agencies. There are already Open Source Intelligence tools that do just this.

Thus it is necessary to use threat modeling and carefully consider use cases based on activity against the proper tools and connections.

Whonix's Anonimity Mode Levels stand as a nice spectrum in approaching things this way - each Mode applicable to different types of sitatuations - and with Qubes, we can build AppVMs respective to each Mode for these varying situations.

### Anonimity Mode use-cases

Let's inspect the Whonix's Documentation page titled ["Tips on Remaining Anonymous"](https://www.whonix.org/wiki/DoNot#Mix_Anonymity_Modes).

Whonix's documentation is careful to define the clear differences between Anonymity and Pseudoanonimity, which is key factor in operational security with this set set up:

> - __Anonymous connection__: A connection to a destination server, where it has no ability to discover the origin (IP address / location) of the connection, nor to associate any identifier with it.
> - __Pseudonymous connection__: A connection to a destination server, where it has no ability to discover the origin (IP address / location) of the connection, but it can be associated with an identifier.

The key here is not to solely think about things like originating IP, but also unique identifiers used in tracking cookies and other technology. There is also the obvious act of logging into an account with a username and password turning what could have been an "anonymous" connection into a technically "pseuudonymous" one - it is a connection that does not reveal your actual originating IP, but _does_ reveal your user information.

A poigniant quote from a developer of the now defunct Liberte Linux drives the point a bit further:

> "I have not seen a compelling argument for anonymity, as opposed to pseudonymity. Enlarging anonymity sets is something that Tor developers do in order to publish incremental papers and justify funding. Most users only need to be pseudonymous, where their location is hidden. Having a unique browser does not magically uncover user's location, if that user does not use that browser for non-pseudonymous activities. Having good browser header results on anonymity checkers equally does not mean much, because there are many ways to uncover more client details (e.g., via Javascript oddities)."

We get the point! This is where the different Whonix Mix Anonimity Modes come into play:

> Mode 1: Anonymous User; Any Recipient

    - Scenario: Posting messages anonymously in a message board, mailing list, comment field, forum and so on.
    - Scenario: Whistleblowers, activists, bloggers and similar users.
    - The user is anonymous.
    - The real IP address / location stays hidden.
    - Location privacy: The user's location remains secret.

> Mode 2: User Knows Recipient; Both Use Tor

    - Scenario: The sender and recipient know each other and both use Tor.
    - Communication occurs without any third party being aware of this activity or having knowledge that the the 
      sender and recipient are communicating with each other.
    - The user is not anonymous. [5]
    - The user's real IP address / location stays hidden.
    - Location privacy: The user's location remains secret.

> Mode 3: User Non-anonymous and Using Tor; Any Recipient

    - Scenario: Logging in with a real name into any service like webmail, Twitter, Facebook and others.
    - The user is obviously not anonymous. As soon as the real name is used for the account login, the website 
      knows the user's identity. Tor can not provide anonymity in these circumstances.
    - The user's real IP address / location stays hidden.
    - Location privacy. The user's location remains secret. [6]

> Mode 4: User Non-anonymous; Any Recipient

    - Scenario: Normal browsing without Tor.
    - The user is not anonymous.
    - The user's real IP address / location is revealed.
    - The user's location is revealed.

Think of these as use cases based on threat models, but also - in the context - as VMs. These scenarios can work quite well paired with respective AppVMs on a QubesOS system. For instance - where a Mode 1-based VM is a generated AppVM titled `Mode-1` using the `Whonix-15` template-VM.

But we can do one better here - we can get a freshly isntalled SubesOS system off the ground with an AppVM for each of these Modes using infrastructure management via Salt.

### Generating the AppVMs with Salt

