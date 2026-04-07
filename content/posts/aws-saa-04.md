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
If you ran the same instance for only 13 hours in the month, you would pay $1.95 (0.15 * 13). On-Demand pricing is best used for unpredictable workloads that vary dramatically both in compute requirements, and uptime of resources.

# Reserved Instances
With the reserved instance payment option you commit to holding a specific number of a specific type of instance in a specific region for a specific ammount of time, either 1 year or 3 years. AWS advertises that you can save over 70% when going with this payment model over the On-Demand payment model. Let's say for example that your workload was an in-memory database, soyou choose a Memory Optomized instance such as `r7g.medium`. At the time of writing that is an instance that has 1vCPU and 8GiB of memory. The On-Demand cost is currently $0.0536/hr and the Reserved cost is $0.0354/hr. If you used the On-Demand model that would cost $469.54 over the corse of a year. The Reserved pricing for the exact same instance would cost $310.10, a difference of over $150.

The "catch" with reserved instances, is that you are locked in. If your needs change mid-year, you are still required to pay for the instance, even if you don't use it. It's essentially a contract with AWS. On-Demand pricing doesnt lock you in like that, you can stop the instance at anytime an only pay for what you need, but as you can see, it is much more expensive.

Think of it like this, with Reserved Instacnces you sign an agreement with the Hertz rental car company to rent 1 (one) 2024 Toyota Corolla from their Chicago Airport location. Hertz agrees to this and gives you a discounted rate and promises that a 2024 Corolla will always be available. If you live in Chicago and need a car, no problem, just rent it from the airport location, and you get the discounted rate. But what if you need a pickup truck to move some furnature? Or a second car for a friend to borrow? Or you travel to Philadelphia and need a car there? In all these cases, your reserved Corolla in Chicago isn't what you need, and so you'll have to pay the regular (On-Demand) price.

This model works best for long lived stable workloads, such as databases; the type of thing that will always be on, and doesn't change often.

# Savings Plan
The Savings Plan payment model is kinda like the Reserved Instance, but is a bit more flexable. With the reserved instances you selected a specific instance type in a particular availability zone and were locked in for the 1 or 3 year term. With the Savings plan, you only commit to a certain amount of money spent, and can spend it on any instance type in any availability zone.

To continue the rental car analogy, the savings plan is like a Hertz gift card. You commit to spending $1,000 this year and load that money on the card. You can then spend that money on any car or truck at any Hertz location anywhere. When you do, you still get a reduced price, just like with the Reserved Instances.

Savinngs Plan works best for workloads that are long-lived, but may need flexability in the compute resources, such as microservices that you may still be optimizing.
