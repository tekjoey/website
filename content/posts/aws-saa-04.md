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

In my last post I introduced what an EC2 instance was and glossed over the different purchasing options that exist. I wanted to devote an entire post to this topic, since there are a whole bunch of options, and it can get overwhelming quickly.

# On-Demand
The easiest option to understand is the On-Demand instance type. With this type, you are simply charged per second (with a 60-second minimum), often expressed as an hourly rate for the amount of time that an instance is running. So if the per hour rate is $0.15 per hour, and the instance is running for 100 hours in the month, you pay $15 (0.15 * 100).
If you ran the same instance for only 13 hours in the month, you would pay $1.95 (0.15 * 13). On-Demand pricing is best used for unpredictable workloads that vary dramatically both in compute requirements, and uptime of resources.

# Reserved Instances
With the Reserved Instance payment option, you commit to holding a specific number of a specific type of instance in a specific region for a specific amount of time, either 1 year or 3 years. 

Let's say for example that your workload was an in-memory database, so you choose a Memory Optimized instance such as `r7g.medium`. At the time of writing that is an instance that has 1vCPU and 8GiB of memory. The On-Demand cost is currently $0.0536/hr and the Reserved cost is $0.0354/hr. If you used the On-Demand model that would cost $469.54 over the course of a year. The Reserved pricing for the exact same instance would cost $310.10, a difference of over $150.

The "catch" with reserved instances, is that you are locked in. If your needs change mid-year, you are still required to pay for the instance, even if you don't use it. On-Demand pricing doesn't lock you in like that, you can stop the instance at any time and only pay for what you need, but as you can see, it is much more expensive.

Think of it like this, with Reserved Instances you sign an agreement with the Hertz rental car company to rent 1 (one) 2024 Toyota Corolla from their Chicago Airport location. Hertz agrees to this and gives you a discounted rate and promises that a 2024 Corolla will always be available. If you live in Chicago and need a car, no problem, just rent it from the airport location, and you get the discount rate. But what if you need a pickup truck to move some furniture? Or a second car for a friend to borrow? Or you travel to Philadelphia and need a car there? In all these cases, your reserved Corolla in Chicago isn't what you need, and so you'll have to pay the regular (On-Demand) price.

This model works best for long-lived and stable workloads, such as databases; the type of thing that will always be on and doesn't change often.

# Savings Plan
The Savings Plan payment model is kind of like the Reserved Instance but is a bit more flexible. With the Reserved Instances you selected a specific instance type in a particular region and were locked in for the 1- or 3-year term. With the Savings plan, you commit to a certain amount of money spent per hour in a 1- or 3-year period and can spend it on (mostly) any instance type in any availability zone. The level of flexibility depends on a few factors, but the flexibility is always there. Importantly, if you don’t use the full amount, you still pay for it.

To continue the rental car analogy, the savings plan is like a Hertz gift card. You commit to spending $200 a day this year and load that money on the card. You can then spend that money on any car or truck at any Hertz location anywhere. When you do, you still get a reduced price, just like with the Reserved Instances.

Savings Plans work best for workloads that are long-lived, but may need flexibility in the compute resources, such as microservices that you may still be optimizing.

# Spot Instances
Spot Instances are Amazon's way of keeping all their servers running at full capacity, even during times of low demand. Since they run on spare capacity that no one actively needs, AWS discounts it heavily. However, once demand picks up or AWS needs to rebalance their infrastructure, they will interrupt your instance after a two minute warning. 

In the rental car analogy, let's say Hertz allows you to rent any idle car for a steep discount. As long as they have the spare inventory, you can rent the car for the price you set, but once someone comes in who is willing to pay the full price, or has a reservation, you are asked to return the car.

Spot instances are best used for workloads that can handle interruptions such as batch processing jobs or testing servers.

# Capacity Reservations
Capacity Reservations are not a pricing model, but they are a related topic and often are used in concert with Savings Plans and Reserved Instances. A Capacity Reservation is just like any other reservation, where you reserve a specific number of instances in an availability zone for a time period that you can set. So maybe you are launching a new web service, and want to ensure


# Dedicated Instances
Dedicated Instances are EC2 instances that run on hardware dedicated to your account. This is primarily a regulatory or compliance need, not really a technical one. An important point is that while the hardware is dedicated to your account, it can change. You aren’t guaranteed the same hardware every time you start & stop your instances, and you cant choose specifically which server an instance gets started on, but you are guaranteed that no one else will have any workloads on that hardware. 

# Dedicated Hosts
If you want (or need) to be able to choose the specific hardware, you will want Dedicated Hosts. This is the most expensive option, because AWS is giving you pretty much full control over an entire server. This is primarily used for applications that are billed based on physical CPU socket count as no other EC2 option exposes the number of physical CPU sockets. Dedicated Hosts also support host affinity, meaning you can schedule an EC2 instance to run on a particular host, and stay running on only that host.
