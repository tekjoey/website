---
title: 'TrueNAS Migration'
date: '2024-12-03'
tags: ["homelab", "alpha-blog"]
# author: ["Me", "You"] # multiple authors
draft: false
UseHugoToc: true
editPost:
    URL: "https://github.com/tekjoey/website/blob/main/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
# description: "This is the description"
# disableHLJS: true # to disable highlightjs
# disableShare: false
# disableHLJS: false
# hideSummary: true

# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page

---
> *This post was originally posted on a Wordpress blog that I had and forgot about. It is being posted here for completeness of my homelab journey. It was originally posted in early 2023.*

***

*This is the first in a series of articles detailing my 2024 homelab update.*

I have been running my current homelab off an old 2013 or 2015 iMac I got from my grandfather, as well as a repurposed gaming PC I built in 2020. The iMac has been my application server (using Docker and Docker Compose to manage the applications), and the gaming PC has been my storage server, running TrueNAS CORE, which you may know is FreeBSD based. Nothing wrong with FreeBSD per se, but I am rather partial to Debian, so when I saw that TrueNAS SCALE was Debian-based, I knew I wanted to switch. Additionally, the 8GB of the iMac was starting to be a limiting factor, and I also wanted to do things “more correctly” like using a hypervisor, Kubernetes, and more Infrastructure-as-Code (IaC).

I decided that the first thing to do was to upgrade my storage server, because once I got that taken care of, I could use it as a storage target. The main problem with my current computer was that it only had space for two HDDs, and I had three, so one was just propped up inside the case. It is a NAS-specific drive, but still, I wanted to give it a better life.

## Initial Testing and Officelabing
The first step in any good upgrade path is choosing the new system and validating that it works for your given needs. My needs were simple:

- Supports Debian
- Have at least 3 spaces for hard drives.

Since Debian is over twenty years old, my first point was pretty much taken care of. As for the second, my day job is as desktop support for two departments at a large university, so there is a pretty steady flow of used computers that we get rid of. After getting the approval of my manager, I choose a Dell Precision T1700. It had spaces for four drives, and had support for 32Gb of non-ECC RAM.

I took a few old hard drives we had lying around at the office, loaded up TrueNAS SCALE, and played around with it in my spare time for a few months. I practiced exporting and importing pools, poked around the UI, and generally got used to the, in my opinion, better interface that SCALE offers over CORE.

Then school let out and the summer slump of my university job hit. Add to that, my wife was going to be out of town for a week, and I had plenty of time to bring this new computer home and initialize my new storage server. Things had been going just fine in my test environment, so I was excited to finally get that third hard drive properly mounted and protected.

## DEGRADED
Ever since I set up my homelab, I had been making backups (some to BackBlaze B2, some to AWS Glacier). Given that, I wasn’t too worried about my data being lost. Worst case scenario, everything breaks and my data is available in the cloud, but not locally accessible.

So I brought this new-to-me computer home, exported my current ZFS pool and shut down my gaming-turned-storage computer. I connected my three hard drives and an SSD for the boot drive to the Precision and booted into TrueNAS SCALE. It installed wonderfully, and I was met with the new dashboard I had come to like. I imported my pool, which seemed to go well until something caught my eye…

`Pool Status: DEGRADED ⚠️`

“Degraded? How could that be?” I hadn’t had any errors (ever) with TrueNAS before, so I wasn’t sure the severity of the word “degraded”. I clicked on my pool information and saw that one of my three drives had not been recognized [^1]. Thankfully, TrueNAS uses ZFS and I had a RAIDZ1 VDEV, so no data was lost. But the health was obviously “degraded”.

Perplexed by this, I exported the pool again and loaded my three drives back to my old server with CORE.

`Pool Status: Healthy ✅`

My old installation was able to read the drive, so it wasn’t a problem with the drive? I ran a short S.M.A.R.T. test on all the drives just to be sure, and it came back with no errors found. Weird.

Thus precipitated a two-day trek through forum posts. What was supposed to be a quick drive switch, turned into a troubleshooting nightmare where nothing would work. One of the first things I found was the 3.3v issue people were having with shucked drives. Mine weren’t shucked, but I gave it a shot. Nothing. I made sure the Precision’s BIOS was up-to-date, it was. The “faulty” drive was discoverable to both my old CORE installation, and to my MacBook (via a USB dock), but not to this Dell Precision. I checked the BIOS of the Dell and intermittently saw only two drives or all three drives show up (usually just the two). I was besides myself.

After two days of troubleshooting, I gave up with that computer and brought home a second. This was a Dell OptiPlex 9020. I could only get 16GB of non-ECC RAM, but it had the same 4 HDD bays as the Precision.

`Pool Status: DEGRADED ⚠️`

Degraded again. On a different system entirely with, presumably, a different enough BIOS. The OptiPlex BIOS also failed to register all three drives.

“This is why people have imposter syndrome.” I thought to myself. “Because sometimes they really don’t know what is going on…”

At this point, I had accepted that I was going to have to use the same hardware, but I at least wanted the HDD bays of the Dell pre-built. I carefully dismantled first the pre-built, and then my old gaming/storage computer and transferred the motherboard. Alas, while my standard ATX motherboard fit, the front-panel connectors are some Dell-proprietary connector. I could probably make it work, but I was so discouraged at this point that I just rebuilt my old setup, this time with TrueNAS SCALE. My third HDD is now is a slightly better position, laid flat on the power supply.

## Lessons Learned
Another good exercise after any kind of deployment or upgrade is to look back and see what could have gone better and what information can we carry into the future.

### The good
While I didn’t exactly plan for downtime, the timing worked out that I would be able to take down my servers for an extended period of time, and would be able to devote sufficient time to fixing any problems. Additionally, I did leave my initial TrueNAS CORE installation intact, so that I could revert to it in the case of a failure (which I ended up having to do). Lastly, my troubleshooting skills came in handy when I had to nail down exactly what the problem was.

### The bad
I didn’t mention this earlier, but one thing I failed to do was to make a backup immediately before I started the swap. My backup schedule was once weekly, and I started this project on a Wednesday, so I could potentially have lost ~4 days of data. In my scenario (where I am the main user and not much data changes) this was acceptable, but part of the purpose of my homelab is learning industry best practice, and “backups before migration” is definitely a best practice.

### Looking forward
I don’t know the next time that I will do a migration like this. It’s very probable that the hardware I have will stay the same for a while, with the possible exception of a new case to hold more drives (maybe an all SSD pool!). Regardless, though, next time I do anything major with my TrueNAS Server, I’ll be sure to take a backup of at least all my critical files, even if I can’t wait to play sysadmin again.

[^1]: The drive in question was a TOSHIBA N300 4TB NAS drive, model HDWG440. The two drives that were recognized were TOSHIBA N300 4TB NAS drives, model HDWQ140. Details, details…