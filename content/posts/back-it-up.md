---
title: 'Back It Up'
date: '2025-12-03'
tags:
    - backups
categories:
    - Homelab
draft: true
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
Backups are like oral hygene. Everyone knows it's important and most people are doing *something*, but very few people are actually putting in the efort to do it completely.

They've always been on my TODO list, but other things have always caught my attention nd distracted me from this all-important endevor. Plus, I was honestly kinda scared to dive into the proper way of doing docker volumes and datbae backups; everything is working fine for now, why change it?

I recently had a scare in my homelab that made me re-think how I have things archetected and how effective my backups actually are. This is a tale as old as time; the problem doesnt get fixed untill it NEEDS to get fixed. Luckally for me, no data was lost, and I'm on track for some major improvements!

# The Incident

I run a Nextcloud instance, and due to some poor configuration on my part I thought I had corrupted some data.
I was playing around with some MINIO S3 buckets (a story for another time) and I mapped one to my Nextcloud account as an external device. I wasnt using it for anything, i just saw it as an option and wanted to play around.
Then, I was cleaning up said buckets and must have deleted the access keys that I used to map it to Nextcloud. Instead of simply giving an error stating "failed to connect to S3 bucket", Nextcloud apparently completely errors out and displays an empty account.
I was able to log in no problem, but none of my files were displayed (the screen just said "no files in here"), none of my calendars or contacts loaded; I seriously thought I had (somehow) bricked my instalation! I was beside myself because I had no idea what I could have done to brick the application. No major changes have happened, no updates, no re-configuration, nothing.
In a panic, I checked the folder I have mounted in the container that actually stores all the data and ...all the files were there.
That's strange, I wonder why Nextcloud isnt recognizing them? As I said, databases scared me, so I had never gotten around to actually backing it up, but I did have a backup of the entire virtual machine, that should work!

It did not.

I then tried to roll back the dataset where I store the data files (using ZFS on TrueNAS). That also didn't work. I contemplated downloading the files from BackBlaze, and just starting over from scratch, but then I thoght to check the server logs. Thats where I saw some error about an S3 failure.
Thankfully I was still able to access that configuration, I delete dit, and everything came back up.

Weird.

But it got me thinking, I didn't have a solid process for restoring from a failure. I *did* have backups, but I didn't have a solid process.
