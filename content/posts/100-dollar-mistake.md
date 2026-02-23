---
title: 'How I spent nearly $100 on useless API calls'
date: '2026-02-18'
tags:
    - homelab
categories:
    - 
draft: true
# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
description: "I accidentally increased by cloud spending by more than 4,500%!"
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

I have a modest homelab, only a few hundred gigabytes of content that doesn't change often. Most of the content is not really that important, but I back it up anyways because, hey, storage costs are pretty low. I use BackBlaze B2, which is about $6/TB, which I am easily under. So you'll image my surprise when my February bill came from BackBlaze, and it was for $103.91.

Yup, I went from spending about $2.50, to over one hundred dollars in about a month. How?

My first though was storage increase. At the time I didn't really put together that that would mean I would have to be storing almost... 17TB of data, and I don't even own enough drives to store that much at home, much less in the cloud. Nevertheless, this was my first, panicky, thought, so I logged into the management console and started investigating. I dug through the buckets and directories, looking for high numbers. I found that my uptime-kuma database was using about 900GB, which is ridiculously high. Drilling down I discovered that, actually, that 900GB was aggregated accross all the file versions of that database. When I set up BackBlaze I had enabled file versioning and set no cap. I had also, somewhat recently started backing up everything every hour, because that sounded like a good idea. Lastly, my understanding of BackBlaze's versioning scheme was incorrect, I thought that they only stored the difference between the versions, but instead they actually store an entire seperate copy, if the file has changed. Uptime Kuma's database includes all the metrics for each node it tracks, so it's database does change frequently. It was currently sitting at about 900mb on my server. So, every hour for the past month, I had been backing up this ~1GB file. Everytime the backup ran, the file had changed, so BackBlaze made an entirely new 1GB file version. You can see how this could esclate. Checking around there were a bunch of other log files that were unneccarily large. I immediately turned off versioning for all my buckets. I never really *needed* it, it was just a nice to have, but not if it was going to cost me $100!

I thought I had solved the problem but I decided to check my billing history, just to see when it started. 2 months ago I saw my normal ~400GB storage cost, costing me about $2.50. Then last month it was increased to ~$14, and about 600GB. Then this past month, the full ~1TB, costing me $100. The storage increase was beause of the hourly backups, that part made sense. But the final cost didn't align, and it was at this this point that i remembered that BackBlaze only charged $6/TB. I clicked on the most recent bill and looked at the details. Class C transactions cost me about $95. Expanding that option I saw that the API call `b2_list_file_names` accounted for all of that.

$95 for an API call. I looked up what the API call does, and as the name implies, it lists out the files in a bucket. I assume this is part of the process when TrueNAS backs up the files, it first needs a list of the existing file structure, but the normal way to do that is to issue one API call for each file and folder. I started backing up once an hour about a month ago, so thats where the $14 then the $100 came from. There's an option in TrueNAS's Cloud Backup jobs to enable `--fast-list`. I learned that this returnes 1,000 paths at once, and only sacrifices a bit of CPU time and memory.

So of course I immediatly enabled `--fast-list` on all my backup jobs. I then looked through BackBlaze's settings looking for alerts. I had wildly over spent, and was never alerted, why was that?

Because I hadent enabled them, dummy. I *had* enabled storage alerts ("Send me an email if I spend more than $5 on storage"), but for Class A, B, and C transactions I had selected "No Cap".

So of course I immediately enabled $5 caps for all of them, just to get started.

To be absolutely clear, none of this is BackBlaze's, or even iXsystems's (makers of TrueNAS), fault. I am a happy customer of each and will continue to each both. This was a case of an over-confident, under-researched guy doing some things and not checking on the results. A lesson to learn from, to be sure. So take this as a warning, check your cloud costs, especially when you make major changes like I did. Turn on alerting for all categories, even ones you dont think you'll use.
