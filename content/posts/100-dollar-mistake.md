---
title: 'How I spent nearly $100 on useless API calls'
date: '2026-02-23'
tags:
    - homelab
    - mistakes
    - backups
categories:
    - Homelab
draft: false
# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
description: "I accidentally increased by cloud spending by more than 4,000%!"
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

I have a modest homelab, only a few hundred gigabytes of content that doesn't change often. Most of the content is not really that important, but I back it up anyways because, hey, storage costs are pretty low. I use [BackBlaze B2](https://www.backblaze.com/), which is about $6/TB, which I am easily under. So you'll image my surprise when my February bill came from BackBlaze, and it was for $103.91.

Yup, I went from spending about $2.50, to over one hundred dollars in about a month. How?

My first though was storage increase, because that was the main metric I think about. At the time I didn't really put together that that would mean I would have to be storing almost... 17TB of data, and I don't even own enough drives to store that much at home, much less in the cloud. Nevertheless, this was my first, panicky, thought, so I logged into the management console and started investigating. I dug through the buckets and directories, looking for high numbers. I found that my uptime-kuma database was using about 900GB, which is ridiculously high. Drilling down I discovered that, actually, that 900GB was aggregated accross all the file versions of that database. 

When I set up BackBlaze I had enabled file versioning and set no lifecycle policy, meaning every version was kept forever. I had also, somewhat recently, started backing up everything every hour, because that sounded like a good idea. Additionally, my understanding of BackBlaze's versioning scheme was incorrect, I thought that they only stored the *difference* between the versions, but instead they actually store an entire seperate copy, if the file has changed. 

Uptime Kuma's database includes all the metrics for each node it tracks, so it's database does change frequently. It was currently sitting at just under 1GB on my server. So, every hour for the past month, I had been backing up this ~1GB file. Everytime the backup ran, the file had changed, so BackBlaze made an entirely new 1GB file version. You can see how this could esclate. Checking around there were a bunch of other log files that were unnecessarily large. I immediately turned off versioning for all my buckets. I never really *needed* it, it was just a nice to have, but not if it was going to cost me $100!

I thought I had solved the problem but I decided to check my billing history, just to see when this price increase started. 2 months ago I saw my normal ~400GB storage cost, costing me about $2.50. Then last month it was increased to ~$14 and about 600GB. Then this past month, the full ~1TB, costing me $103. The storage increase was beause of the hourly backups and versioning, that part now made sense. But the final cost didn't align, and it was at this this point that I remembered that BackBlaze only charged $6/TB, meaning I was being charged about $97 for something other than storage. I clicked on the most recent bill and looked at the details. Class C API transactions cost me about $97. Expanding that option I saw that the API call `b2_list_file_names` accounted for all of that and that it has been called over 1 billion times.

$97 for over 1 billion API calls. I looked up what the API call does, and as the name implies, it lists out the files in a bucket. I assume this is part of the process when TrueNAS backs up the files, it first needs a list of the existing file structure, but the default way that it is done is to issue one API call for each file and folder. However, all the advice I found online recomended enabling `--fast-list` in the settings for each Cloud Backup job on TrueNAS. This reduces the number of API calls, while increasing CPU usage slightly because it recieves multiple file paths (up to 1,000) in one API call.

So of course I immediatly enabled `--fast-list` on all my backup jobs. I then looked through BackBlaze's settings looking for alerts. I had wildly over spent, and was never alerted, why was that?

Because I haden't enabled them! I *had* enabled storage alerts ("Send me an email if I spend more than $5 on storage"), but for Class A, B, and C transactions I had selected "No Cap".

So of course I immediately enabled $5 caps for all of them, just to get started.

To be absolutely clear, none of this is BackBlaze's, or even iXsystems's (makers of TrueNAS), fault. I am a happy customer of each and will continue to use both. This was a case of an over-confident enthusiast who started messing with things he didn't fully understand at the time. 

So take this as a warning (I sure am), check your cloud costs, especially when you make major changes like I did. Turn on alerting for all categories, even ones you don't think you'll use.
