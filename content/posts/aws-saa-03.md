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
This is my first post regarding AWS EC2. This post is an introduction to what EC2 is and then some basic set up steps to get it up and running.

# EC2 Introduction
Virtual Computing is one of the cornerstones of modern computing. Virtualization allows for secure and efficient use of the massive resources that modern servers can provide. 
AWS offers a variety of virtualized computing services, but the first and most basic is Elastic Cloud Compute (EC2).

EC2 was one of the first services that AWS launched with back in 2006/07. It allows you to run a virtual server in the cloud at the click of a button. Even though I understand how it all works behind the scenes, it's still magical to be able to click-click-click and have a virtual server boot up in a data center somewhere.

The EC2 service is actually a collection of distinct, yet related, services. They are:
- EC2 Instances. These are the actual virtual machines.
- EBS (Elastic Block Storage). These are the drives that get attached to an EC2 Instance to store data.
- ELB (Elastic Load Balancer). This is a service to distribute incoming network traffic between multiple VMs. More on this in a later post
- ASG (Auto-Scaling Group). This is a service that, once configured, will automatically create or destroy new VMs as neccesssary. More on this also in a later post.

## Setting Up an EC2 Instance

Every computer needs 4 things; a CPU, memory, storage, and a set of instructions (in this case, an operating system). An EC2 instance is no different. When you begin to launch the instance, one of the first questions you'll be asked is which Amazon Machine Image (AMI) you want to launch with.

In its most basic state, an AMI is just an opperating system. But unlike an OS, an AMI can include prconfigured software. Amazon has a whole bunch of options, from basic OSs like Linux, Windows, and even macOS, to network appliances like firewalls or Kubernetes nodes. 
They also have an image builder, so if you need a fully custom AMI, you can build that yourself.

Once you have the AMI selected, it's time to select the Instance type, which combines the CPU and memory. AWS has a number of instance familes, each of which is designed for specific types of tasks
- General Purpose: These offer a balance of CPU and memory. As the name implies, these are the go-tos for the average computing needs.
- Compute Optimized: These offer more CPU cores with less memory, as compared to the general purpose options. These instances excell at high volume work such as batch procesing, or media transcoding.
- Storage Optimized: These offer millions of low-latency, random I/O operations per second, perfect for time-sensitive opperations such as databases, or certain data processing applications.
- Memory Optimized: these offer more memory than the general purpose options, with fewer CPU cores. These are great for in-memory databases, or certain analytics programs.
- HPC Instance: These instances are best suted for high performace tasks such as complex model simulations or deep learning
- Accelerated Computing: These instances have hardware accelerators, or co-processors, to perform functions such as precise floating-port opperations or graphics processing more efficiently.

Each one of these familes have many pre-configured options. You can browse through them at [https://instances.vantage.sh/](https://instances.vantage.sh/)

Lastly, after youve selected the AMI (operating system) and the instance type (CPU & RAM), your VM will need durable storage. That's where EBS comes in. It provisions and manages all the virtual drives in your account. Obviously your EC2 Instance needs a main/root drive for the opperating system, but you can also add more drives, just like you can in a physical server.

EBS Volumes come with a few different options, which all boil down to "how fast and how performant do you want your drive". The options span the gamut from infrequent cold storage (HDD-based st1) to general purpose SSDs (gp3) to maximum performance for critical databases (io2 Block Express).

After selecting the AMI, Instance type and creating an EBS volume, you *technically* have everything you need. However, the VM will be isolated because we havent said anythig about networking yet. We'll talk later about VPCs and network subnets, for now those can just be set to defaults. However, one thing we do need to talk about is security groups!

Security Groups are collections of firewall rules. Their main advantage is the fact that they are a seperate entity from the EC2 instance, so you can apply the same set of firewall rules to multiple EC2 instances. Just like any firewall ruleset you can specifiy incoming or outgoing ports and IP addresses. When a security group is first created, it has a default rule to allow port 22 (SSH) from any IP address, and to allow all outbound ports/IPs from the VM. You can (and obviously, should) change this to suit your needs. This is the default just to get you up and running via SSH.

After this your EC2 instance is good to go! As with anything, there are several additional options for advanced usage such as joining a domain, changing shutdown behavior, or any other specific requirments you may have. An option that I only recently discovered was the "User Data" section. Here you can add a script to run durring the first boot of machine. This can be used to update the package repository and install specific programs, or anything else your workflow requires on first boot. Note: this script ONLY runs on first boot, every boot afterwards will be normal and will not execute this script.

Well that's enough about EC2 for now, but that's not all there is to say! Stay tuned for more on this topic!
