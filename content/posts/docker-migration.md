---
title: 'Docker Migration'
date: '2024-12-04'
tags:
    - migration
    - docker
    - docker-compose
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
*This is the second in a series of articles detailing my 2024 homelab update.*

As I said in my last post, my homelab has consisted of an application and storage server, with the application server being a 2013 iMac running Ubuntu Server 20.04 LTS. This has worked out fine, but I am running up against the 8GB RAM limit of the iMac, especially now that I have deployed Immich as photo storage. I also wanted to run more than just containers (Local DNS, Home Assistant, Kubernetes) so I wanted a hypervisor OS to be able to easily manage all of that. I've heard a lot about Proxmox, and since it is basically just Debian plus QEMU I figured it would be a nice easy choice. Plus, it's free and open source!

Similar to my TrueNAS upgrade, I wanted to first get a feel for the Proxmox installation process and understand how to use it before I deployed it for real. Again, similar to my TrueNAS upgrade, I found a spare computer from work and loaded up Proxmox.

The installation process was simple and once I got it setup, I found the UI fairly intuitive, if a bit dated.