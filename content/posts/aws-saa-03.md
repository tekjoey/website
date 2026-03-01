---
title: 'AWS Solutions Architect Associate - Pt3 (EC2)'
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
description: "Part 3 of my journey to recieving my AWS Certified Solutions Architect Associate certification. Here, I begin talking about EC2."
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

Virtual Computing is one of the cornerstones of modern computing. Virtualization allows for secure and efficient use of the massive resources that modern servers can provide. 
AWS offers a variety of virtualized computing services, but the first and most basic is Elastic Cloud Compute (EC2).

EC2 was one of the first services that AWS launched with back in 2006/07. It allows you to run a server in the cloud at the click of a button. Even though i understand how it all works behind the scenes, it's still magical to be able to click-click-click and have a virtual server boot up in a data center somewhere.

The EC2 service is actually a collection of different, yet related, services. They are:
- EC2 Instances. These are the actual virtual machines.
- EBS (Elastic Block Storage). These are the drives that get attached to an EC2 Instance to store data.
- ELB (Elastic Load Balancer). This is a service to distribute incoming network traffic between multiple VMs.
- ASG (Auto-Scaling Group). This is a service that, once configured, will automatically create or destroy new VMs as neccesssary. More on this later.

Every computer needs 3 things; a CPU, memory, storage, and a set of instructions (in this case, an operating system). An EC2 instance is no different. When you begin to launch the instance, one of the first questions you'll be asked is which Amazon Machine Image you want to launch with.
In its most basic state, an AMI is just an opperating system. But unlike an OS, an AMI can include prconfigured software. Amazon has a whole bunch of options, from basic OSs like Linux, Windows, and even macOS, to network appliances like firewalls or kuberneties nodes. 
They also have an image builder, so if you need a fully custom AMI, you can build that yourself.

Once you have the AMI selected, it's time to select the Instance type, which combines the CPU and memory. AWS has a number of instance familes, each of which is designed for specific types of tasks
- general Purpose: These offer a balance of CPU and memory. A sthe name implies, these are the go-tos for the average computing needs.
- Compute optomized: These offer more CPU cores with less memory, as compared to the general purpose options. These instances excell at high volume work such as batch procesing, or media transcoding.
- storage optomized: These offer millions of low-latency, random I/O operations per second, perfect for time-sensitive opperations such as databases, or certain data processing applications.
- memory optomized: these offer more memory than the general purpose options, wih fewer CPU cores. These insatnces are for memory heavy workloads, such as in-memory databases, or certain analytics program.
- HPC instance: These instances are best suted for high performace tasks such as complex model simulations or deel learning
- accelerated computing: These instances have hardware accelerators, or co-processors, to perform functions such as precise floating-port opperations or graphics processing more efficiently.

Each one of these familes have many pre-configured options. You can browse through them at [https://instances.vantage.sh/](https://instances.vantage.sh/)

Lastly, each VM needs durrabl storage. That's where EBS comes in. 

