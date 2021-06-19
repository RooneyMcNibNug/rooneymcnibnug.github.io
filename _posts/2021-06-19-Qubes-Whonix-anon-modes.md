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

I love QubesOS. I use it as a daily driver for a lot of things at this point, especially compartmentalized communications. Many people I speak to use Qubes as well, so I often pick their brains about use cases, [threat models](https://www.eff.org/document/surveillance-self-defense-threat-modeling), and what bells and whistles they've set up on their system.

I was speaking with one person in particular about how they manage privacy schemes with AppVMs, and they told me about how they rely on following guidelines put forth by Whonix in their wiki (located on the Whonix Anonymity Modes page). This idea makes a lot of sense to me, and I was eager to try it out.

### Understanding Anonimity Mode levels

To me, a fundamental consideration with these different anonymity modes is data retention. If you are posting something to a public forum with a username connected to your actual identity in one way or another, its good to get into the mindset that you "left a piece of you" somewhere. The internet in constantly mined for user content and  metadata: your contact information, your "likes", your political opinions and ideologies, etc. It is every marketing agency's dream to be able to tie everyone's account details to one another to paint a mosiac of a user's desires - data brokers have big dreams of operating as if they were intelligence agencies. There are already Open Source Intelligence tools that do just this.

Thus it is necessary to use threat modeling and carefully consider use cases based on activity against the proper tools and connections.

Whonix's Anonimity Mode Levels stand as a nice spectrum in approaching things this way. Each Mode is applicable to different types of sitatuations and with Qubes, we can build AppVMs respective to each Mode for these varying situations.

### Anonimity Mode use-cases

Let's inspect the Whonix's Documentation page titled ["Tips on Remaining Anonymous"](https://www.whonix.org/wiki/DoNot#Mix_Anonymity_Modes).

Whonix's documentation is careful to define the clear differences between what should be considered Anonymity and Pseudoanonimity, which is key factor in operational security with this set set up:

> - __Anonymous connection__: A connection to a destination server, where it has no ability to discover the origin (IP address / location) of the connection, nor to associate any identifier with it.
> - __Pseudonymous connection__: A connection to a destination server, where it has no ability to discover the origin (IP address / location) of the connection, but it can be associated with an identifier.

The key here is not to solely think about things like originating IP, but also unique [identifiers used in tracking cookies and other technology](https://coveryourtracks.eff.org/learn) as well as user information. To put it plainly in an example, the act of logging into an account with a username and password turns what could have been an "anonymous" connection into a technically "pseuudonymous" one - it is a connection that does not reveal your actual originating IP, but _does_ reveal your user information.

A poigniant quote from a developer of the now defunct Liberte Linux drives the point a bit further:

> "I have not seen a compelling argument for anonymity, as opposed to pseudonymity. Enlarging anonymity sets is something that Tor developers do in order to publish incremental papers and justify funding. Most users only need to be pseudonymous, where their location is hidden. Having a unique browser does not magically uncover user's location, if that user does not use that browser for non-pseudonymous activities. Having good browser header results on anonymity checkers equally does not mean much, because there are many ways to uncover more client details (e.g., via Javascript oddities)."

I think we get it now. This is where the different Whonix Mix Anonimity Modes come into play:

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

Think of these as use cases based on threat models, but also - in the context - as compartmentalized VMs. These scenarios can work quite well paired with respective AppVMs on a QubesOS system. For instance - where a Mode 1-based VM is a generated AppVM titled `mode-1` using the `whonix-ws` template-VM and `sys-whonix` network-VM.

But we can do one better here - we can get a freshly installed QubesOS system off the ground with this setup. Let's create an AppVM for each of these Modes using infrastructure management via [Salt](https://docs.saltproject.io/en/latest/).

### Generating the AppVMs with Salt

Ever since way back with version R3.1, QubesOS users have been able to [utilize the Salt management engine](https://www.qubes-os.org/doc/salt/) to build/control Templates and AppVMs from the `dom0` level on their system.

This comes in handy for quickly deploying VMs based on an `.sls` file - or ["Salt State File"](https://docs.saltproject.io/en/latest/topics/tutorials/starting_states.html) - configured by a user. I have built a [repo on github with these state/config files for AppVMs](https://github.com/RooneyMcNibNug/qubes-salt-anon-modes) to be built repsective to each one of these Modes.

Here is an example of the `.sls` file for the `mode-1` AppVM:

```yaml
Mode 1: Anonymous User; Any Recipient

create-mode-1-vm
  qvm.vm:
      - name: mode-1
      - preset:
        - template: whonix-ws
        - label: red
        - mem: 3000
      - prefs:
        - template: whonix-ws
        - netvm: sys-whonix
```

"Sitting above" the `.sls` files is a `.top` file that serves as [a deployment file](https://docs.saltproject.io/en/latest/ref/states/top.html) for the grouping of machines here. Here is the `.top` file needed to generate all the Modes-based AppVMs - based on their `.sls` files - from `dom0`:

```yaml
base:
    dom0:
        - mode-1
        - mode-2
        - mode-3
        - mode-4
```

Pretty simple, huh? It is basically just pointing to the `.sls` files and will take the underlying metadata to build each of these AppVMs. So let's get started with pushing this out with these following steps:

- It is best not to copy things _to_ `dom0` with the QubesOS tools to do so, as you would normally do from AppVM to AppVM - this has been [noted as a security risk](https://www.qubes-os.org/doc/copy-from-dom0/). Knowing this, it might be best to clone the repo here and copy the underlying files in a bit more of a manual way here...
- You will want to put your `.top` file somewhere such as `/srv/salt/qubesmodes.top` and your `.sls` fiels in the same directory, like so: `/srv/salt/mode-1-vm.sls`
- Enable the `.top` file from `dom0` terminal with the following command: ```qubesctl top.enable qubesmodes```
- Let's make sure we see the `.top` file we created for this in the output after this ommand, which is used to check enabled `.top` files: ```qubesctl top.enabled```
- If we see it there, we are ready to apply the build of the AppVMs now with the following command: ```qubesctl --show-output --skip-dom0 --all state.highstate```
- Notice that we are getting the verbose output with the added parameters in this command, and also telling the system to not worry about `dom0` states, since we are only focused on building AppVMs in the context here.
- You should now see the VMs populated on your Qubes VM Manager!

### Class of characters

So we have our generated AppVMs based on the Whonix Mixed Anonymity Modes. Now what? Well, we obviously want to start installing some applications to use for each, based on the aformentioned Whonix documentation.

While we really haven't pre-configured applications for use here from the start, we mainly wanted to get a "hull" for each AppVM going - from there on, it is up to you to decide what applications you will use within each Mode. However, it is definitely an option to adjust the Salt files to include these to install automatically as well - Kushal Das goes more into this in [one of his great blog posts](https://kushaldas.in/posts/maintaining-your-qubes-system-using-salt-part-1.html), which can serve as a guide to do this at an earlier stage with the management stack.

In this case, let's have a think at some of the options and apps well reserved for each one of these modes.

Here are some considerations to dabble with:

- Installing [OnionShare](https://onionshare.org/) on `sys-whonix` to use within the `mode-1` AppVm as a tool to send/recive files or quickly host file anonymously via the tor network (this can be a bit more tricky to get set up in a Qubes/Whonix environment compared to a standard system - [this article](https://www.whonix.org/wiki/OnionShare) goes over some of the intricacies and should serve well with setting things up)
- Using [Hexchat](https://hexchat.github.io/) in either `mode-1` for an ephermeral usage, where you will not retain your presence/username in a particular channel, or `mode-3` if there is not a concern about identity but the user wishes some of their connection details to remain somewhat private. (Whonix has a [good guide for this too](https://www.whonix.org/wiki/HexChat))
- Pseudonymize things like emails, form submissions, records requests, etc. by using a dedicated email configuration with a client such as [Thunderbird](https://www.thunderbird.net/en-US/) in `mode 2` or (more likely) `mode-3` - and effectively strip out most actual identifying metadata attached.
- Pseudonymize end-to-end encrypted communications with Signal within your `mode-3` AppVM (you can optionally [use a secondary phone number](https://theintercept.com/2017/09/28/signal-tutorial-second-phone-number/) and configure the application consulting [this guide](https://www.whonix.org/wiki/Signal))
- Scrape data from public sites anonymously with your artisinally-crafted script in the `mode-1` terminal (your mileage may vary!)

### Closing

There are plenty of other ideas you can commit to with this particular compartmentalized approach in QubesOS. It serves as a great way to force a user to be conscious of what is going on in the background when they make a connection, a post, or commit just about any act using the internet, and generally allows them to do so in a way that fits their specific needs in terms of privacy.

For a bit of a meta-joke conclusion, here is s screenshot of me creating this blog post in a `mode-4` AppVM:
<p align="center">
  <img src="/img/qubes_modes_screenshot.png">
</p>

I'd love to hear about your QubesOS schemes. You can reach out to me at [@rooneymcnibnug](https://twitter.com/rooneymcnibnug) on The Bird Site or [email me](https://rooneymcnibnug.github.io/links/) if you want to chat.

Safe surfing! :computer::surfer:
