---
title: '2023 Homelab'
date: '2024-12-02'
tags: ["homelab", "alpha-blog"]
# author: ["Me", "You"] # multiple authors
draft: false
UseHugoToc: true
editPost:
    URL: "https://github.com/tekjoey/website/blob/main/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

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
> *This post was originally posted on a Wordpress blog that I had and forgot about. It is being posted here for completeness of my homelab journey. It was originally posted in early 2023.*

***

I’ve decided to layout what my homelab looks like now because this summer I hope to change some things.


## Storage
My current storage server is one that started life as a gaming computer that I built back in 2020. I had gotten a wild itch to build a computer (my first ever build), that admittedly I didn’t really plan much out. I also didn’t really factor in the fact that I am not a gamer. So after I built it I toyed around with Cities:Skylines, Stardew Valley, and other such games, but the computer largely sat unused (or at least under-used).


Then I discovered homelabing, and suddenly I had a need for networked storage. I purchased three 4TB Toshiba drives and loaded up TrueNAS Core and had myself a fancy little Network Attached Storage server. The computer case I had was really only meant for one hard drive, so two of my hard drives are currently resting in some fairly precarious positions inside the case.


## Applications
What is homelabing without applications? I am currently running Ubuntu Server on an old iMac my grandfather gave me. On that server, I am using Docker as the container engine to run the various applications. Along with that, I am using Docker Compose as my “Infrastructure as Code”. Most of my docker containers use Docker Volumes to store data, but for those that store a “significant” amount (which is a metric I do not have a definition for) I use bind-mounts that connect back to my aforementioned TrueNAS server via NFS.


When I first started on this homelab journey, I went a little wild, deploying every application I came across, whether I actually had a need for it or not. After a while (and my wife’s need for a more present husband), I stopped deploying every container under the sun and settled into actually using those I had a use for, and getting rid of those I don’t. Currently, the ones I use are:


* Nextcloud w/ Colabora Office: Google Drive replacement
* Immich: photo storage
* Authentik: authentication/SSO
* Firefly-III: personal finance management
* Homepage: links and mini dashboard
* Mealie: recipe management
* Paperless-NGX: virtual filing cabinet
* Traefik: load balancer/reverse proxy
* code-server: VS Code environment to access all my docker compose files
* Dozzel: live log viewer
* FreshRSS: RSS feeder. I use NetNewsWire on iPhone to actually read articles
* Homebox: physical inventory management


## Network
I recently upgraded my network gear from the combo-box that Spectrum gave me to a TP-Link OMADA switch and access point with an OPNSense router. I do have three separate VLANS (Trusted, IoT and Guest), but I haven’t actually done any real configuration, so they’re not providing any security benefits.


Furthermore, I am also using Cloudflare DNS as my only DNS service. This means, if I want a DNS record, I must expose the application to the internet. When I was still using Spectrum’s combo box, I tried setting up PiHole as a container, but for some reason I wasn’t able to get it to work. I haven’t exposed every service to the internet, so some services are still referred to by IP:Port and if I didn’t have Homepage, I would definitely forget which ones were which.


## Summer plans
My goals for this summer (2024) are modest, but quite substantial, given the current state of things:


**Proper network setup**
- Firewall Rules to restrict IoT movement/connections and fallout
- Internal DNS server so that every service can have a name and I can stop referencing to IP:Port.
- Route everything with Tailscale (one of the coolest gosh-darn products to ever exist)


**Transfer to “new” servers.** As previously stated, I’m currently running everything off an old iMac. This works, But I’m hitting the limit of what’s possible with its 8GB of RAM. I’ll transition to a more “production” environment by using Proxmox and running everything in Virtual Machines. This will give me the ability to more easily create snapshots/backups and spin up/down resources as needed.


**Transition to use more IaC.** Infrastructure as Code is such a neat concept. The idea that my entire stack, from hardware to application, can be described and deployed via a configuration file, which can then be saved, stored, tracked via change control and all the rest, is such a wonderful idea and I want to be part of it. This will probably start off with just using a bit of Ansible and vanilla Kubernetes, but my eventual goal/dream is to learn NixOS and Terraform/OpenTofu and be able to recreate everything with as minimal intervention as possible. Additionally, this would make it easier to create an “emergency script” so that in the (hopefully very distant) future when I die and my family takes over my homelab and network, they would be able to revert it to a “simple” state, or even clean everything up to dispose of/sell.


Well, that’s about all I have. I’ll have more posts as I preform all the changes I just mentioned. Hopefully I’m able to accomplish all of them.

#SoliDeoGloria