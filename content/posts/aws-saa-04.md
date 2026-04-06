---
title: 'AWS Solutions Architect Associate - Pt4 (EC2)'
date: '2026-02-18'
tags:
    - aws
    - journal
categories:
    - Certifications
draft: true
# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
description: "Part 4 of my journey to recieving my AWS Certified Solutions Architect Associate certification. Here, I begin continue about EC2."
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

In my last post I introduced wan an EC2 instance was, and glosed over the different purchesing options that exist. I wanted to devote an entire post to this topic, since there are a whole bunch of options, and it can get overwhelming quickly.

# On-Demand
The easiest option to understand is the On-Demand instance type. With this type, you are simply charged a per hour rate for the amount of time that an instance is running. So if the per hour rate is $0.15 per hour, and the instance is running for 100 hours in the month, you pay $15 (0.15 * 100).
If you ran the same instance for only 13 hours in the month, you would pay $1.95 (0.15 * 13). On-Demand pricing is best used for workloads that vary dramatically both in compute requirements, and uptime of resources.

# Reserved Instances
With the reserved instance payment option you commit to holding a specific number of a specific type of instance in a specific region for a specific ammount of time. AWS advertises that you can save over 70% when going with this payment model over the On-Demand payment model.
This model works best for long lived workloads, such as databases; the type of thing that will always be on, and doesn't change often.
