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
*This is the second in a series of articles detailing my 2024 homelab update. This post was written about 6 months after the actual events happened, so it will be less of a tutorial, and more of a high-level overview. Next time maybe I'll take notes...*

As I said in [my last post](truenas-migration.md), my homelab has consisted of an application and storage server, with the application server being a 2013 iMac running Ubuntu Server 20.04 LTS. This has worked out fine, but I am running up against the 8GB RAM limit of the iMac, especially now that I have deployed Immich as photo storage. I also wanted to run more than just containers (Local DNS, Home Assistant, Kubernetes) so I wanted a hypervisor OS to be able to easily manage all of that. I've heard a lot about Proxmox, and since it is basically just Debian plus QEMU I figured it would be a nice easy choice. Plus, it's free and open source!

Similar to my TrueNAS upgrade, I wanted to first get a feel for the Proxmox installation process and understand how to use it before I deployed it for real. Again, similar to my TrueNAS upgrade, I found a spare computer from work and loaded up Proxmox.

The installation process was almost as seamless as a usual OS install with one exception; for some reason Proxmox 8.1 did not work with [Ventoy](https://www.ventoy.net/en/index.html), my preferred multi-iso program for flash drives. I would load boot up the computer and choose the Ventoy disk, then choose the proxmox-8-1.iso file, but once it started loading it would say that the boot file was not found and time out. I fixed this by simply flashing the Proxmox ISO straight to a flash drive using [Balena Etcher](https://etcher.balena.io/), but I was a bit bummed that I had to do that. Hopefully Proxmox will fix this issue. After that I was able to go through the graphical installation process, which is fairly straightforward, and ta-da, I have a Type-1 Hypervisor!

From there I was able to play around with the web-interface and generally get to know the system before I deployed it in my homelab.

## Deployment
When I was finally ready to deploy it to my homelab I brought the computer home and reinstalled Proxmox so any configuration I did in testing wouldn't persist. Curently, I only have the one computer, so I wont have a cluster, but at this point, I don't really need that; maybe one day though...

My first attempt was to make a disk image of my iMac that I could just import into Proxmox and connect to a VM.  I tried that by installing the `qemu-img` utility and saving the resulting image on my NAS. It took over a day to finish, so I was happy I was using [NTFY](https://ntfy.sh/) to notify me when it was done. I copied it over to the Proxmox node and tried to import it, but it didn't work