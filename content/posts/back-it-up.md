---
title: 'Back It Up - Part 1'
date: '2025-12-10'
tags:
    - backups
categories:
    - Homelab
draft: false
# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
description: "Time to get serious about backups!"
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
Backups are like oral hygiene. Everyone knows they're important and most people are doing *something*, but very few people are actually putting in the effort to do it completely.

They've always been on my TODO list, but other things have always caught my attention and distracted me from this all-important endeavor. Plus, I was honestly kinda scared to dive into the proper way of doing Docker Volume and database backups; everything is working fine for now, why change it?

I recently had a scare in my homelab that made me re-think how I have things architected and how effective my backups actually are. This is a tale as old as time; the problem doesn't get fixed until it NEEDS to get fixed. Luckily for me, no data was lost, and I'm on track for some major improvements!

# The Incident

I run a Nextcloud instance, and due to some poor configuration on my part I thought I had corrupted some data.
I was playing around with some MINIO S3 buckets (a story for another time) and I mapped one to my Nextcloud account as an external device. I wasn't using it for anything, I just saw it as an option and wanted to play around.
Then, later, I was cleaning up said buckets and must have deleted the access keys that I used to map it to Nextcloud. Instead of simply giving an error stating "failed to connect to S3 bucket", Nextcloud apparently completely errors out and displays an empty account.
I was able to log in no problem, but none of my files were displayed (the screen just said "no files in here"), none of my calendars or contacts loaded; I seriously thought I had (somehow) bricked my installation! I was beside myself because I had no idea what I could have done. No major changes have happened, no updates, no re-configuration (besides the S3 bucket, which I had forgotten about), nothing.
In a panic, I checked the folder I have mounted in the container that actually stores all the data and... all the files were there.
That's strange, I wonder why Nextcloud isn't recognizing them? As I said, databases scared me, so I had never gotten around to actually backing it up, but I did have a backup of the entire virtual machine, that should work!

It did not.

I then tried to roll back the dataset where I store the data files (using ZFS on TrueNAS). That also didn't work. I contemplated downloading the files from BackBlaze, and just starting over from scratch, but then I thought to check the server logs. That's where I saw some error about an S3 failure.
Thankfully, I was still able to access that configuration, I deleted it, and everything came back up.

Weird.

But it got me thinking, I didn't have a solid process for restoring from a failure. I *did* have backups (I'm not a crazy person alright), but I didn't have a solid process. I was backing up the data to an S3 bucket in BackBlaze, but I wasn't backing up the database or the config files. I was backing up the VM, but none of the individual containers. And despite using Docker Compose (as a psudo IaC option), I wasn't using git, so any changes were pretty much permanent.

I decided enough was enough. Let's do this the right way!

# The Plan

So I did what any person would do nowadays, I turned to ChatGPT. It suggested some ideas, I added some others, asked some questions, did some research and finally, this is the plan I've come up with:

- All databases will use Docker Volumes. A script will be created to generate backups of these databases daily. Since I prefer to use Postgres, this can be done while the databae is running with no interuption.
- All other volumes will be NFS mounts from TrueNAS mounted via Docker NFS mounts (not mounted to the host). Since TrueNAS uses ZFS, snapshots of these dataset's can be taken, and the datasets can also be backed up to BackBlaze S2. These snapshots can be programmatically taken through the same script that backs up the databases.
- If a container's volume consists of purely configuration files, those will be bind mounted and versioned with git.
- All environment variables will be placed in `.env` files and encrypted.
- All Compose, `.env`, and other config files will be versioned with git and backed up to GitHub.

This should give me the granular ability that I want. I'll be able to surgically recreate an individual container, instead of affecting all containers at once. Once this is fully implemented, I'm planning on spinning up a test VM and will attempt to actually recreate my production VM.

# The Implementation.

Some containers were easy to get setup with this new system. [Traefik](https://github.com/traefik/traefik), for example, (my reverse proxy), only uses configuration files, no real volume needed. It was already setup with its config files on a bind mounted folder on the host, so it was ready to go immediately. Others, however, are a bit more complicated. Nextcloud for example.

I am using the [LinuxServer.io image](https://github.com/linuxserver/docker-nextcloud) for Nextcloud. This is a wonderful image that has two volumes, `data` and `config`. The `config` volume I had previously setup as a regular Docker Volume, while the `data` volume was bind mounted to a folder on the host that itself was an NFS mounted dataset from TrueNAS. I wanted both of these to be NFS shares, and both to be NFS Docker Volumes, cutting out the middle man and connecting the container directly to the NFS server. This would also fix a minor problem I've had where when the host rebooted, the container would start before the NFS share was mounted, requiring me to manually restart any container that relied on NFS.

To make the `config` volume an NFS share, I would obviously need to export the data from the Docker Volume and move it to the NFS server. That was easy enough to do with the following commands:

```bash
docker exec -it nextcloud tar -cf config.tar /config
docker copy nextcloud:/config.tar ./config.tar
```

The first command create a `tar` file (the Linux version of a zip file) of the config directory. The second command then copies that file out from the container to the host. Both of these commands need to be run while the container is running. After I copied the config.tar file out, I stopped both the nextcloud server and database containers.

I then created two child dataset on TrueNAS under my top-level Nextcloud dataset (the one currently holding just the data). These child datasets were `config` and `data`. I copied the tar file into the `config` dataset, un-tar'ed it, then copied the data into the `data` dataset.

Of course, I had a weird issue with the NFS folders. Since NFS treats each dataset as it's own file system, you can't mount both the parent and child datasets via NFS. I had the datasets `pool/Nextcloud`, `pool/Nextcloud/config` and `pool/Nextcloud/data` all mounted with NFS on the one host. When I transfered the config file into the `config` dataset, TrueNAS registered the increased size in the parent dataset (`pool/Nextcloud`), not the child dataset (`pool/Nextcloud/config`). But navigating through TrueNAS's shell, the config.tar file was nowhere to be found. It was really weird, and caused me even more stress (and lead to a few snapshot rollbacks). I then found [this post](https://forums.truenas.com/t/unable-to-write-to-child-datasets-within-nfs-share/47846) where the poster had the same problem. Thankfully the solution was easy, only share the child datsets. I did this, mounted them to the host, transfered all the relevant data, then unmounted them from the host and edited my Docker Compose files to include two new volumes

```docker compose
services:
  app:
    image: lscr.io/linuxserver/nextcloud
    volumes:
      - nfs_config:/config
      - nfs_data:/data
...
volumes:
  nfs_config:
    driver_opts:
      type: nfs
      o: addr=truenas.mckay.one,rw,nfsvers=4
      device: ":/mnt/pool/nextcloud/config"
  nfs_data:
    driver_opts:
      type: nfs
      o: addr=truenas.mckay.one,rw,nfsvers=4
      device: ":/mnt/pool/nextcloud/data"
```

Alright, cool! Everything should be setup, just a quick `docker compose up -d` and we'll be on our....

502 Bad gateway

Checking server logs, a whole bunch of "unable to write file, permision denied"

What? Why?

Using `docker exec`, I checked the permissions in the container, and all the data and config files were correct.

At this point I was (proverbially) ripping my hair out. I tried a few other things, but nothing panned out. I was a bit desperate, and frantic at this point. I was so close, but I also felt way out of my leauge. So, once again, I turend to ChatGPT (one of these days I've gotta get a local LLM setup...) ChatGPT suggested that TrueNAS was applying `root_squash` to the mounts. I remember reading about that a few years ago. Thankfully, ChatGPT explained it pretty well

>Your container runs as UID 1000, and your files on NFS are owned by 1000:1000, which is good.
>
>BUT you're mounting the NFS share as root (UID 0) inside the container — because Docker mounts volumes as root — and TrueNAS is most likely applying root_squash, which maps root → nobody (UID 65534).
>
>So even though the files look like they're owned by 1000:1000 inside the container, all operations coming from root get mapped to nobody when they hit the NFS server.
>
>This results in:
>- Reads may work
>- Writes fail
>- Ownership appears right inside the container, but NFS silently denies operations
>
>This is exactly what you’re experiencing.

That made sense! Since Linux relies heavely on UIDs and GIDs, the actual number of the user or group is important. UID 0 (the root acount) in the container was being mapped to user 65534 on the TrueNAs system, also known as `nobody`. This `nobody` user does not have access to the `pool/nextcloud/config` dataset on TrueNAS, so write opperations were failing. So all I had to do was explicently map the containers root user to TrueNAS's root user This can be done via the "Maproot User" and "Maproot Group" settings in the Advanced NFS share settings in TrueNAS.

NOW just a quick `docker compose down && docker compose up -d`...

And it works! All of my files are there, as well as all of my contacts, calendars, and settings!

# Conclusion
Well I learned a lot this go-around. I learned about NFS file permissions, NFS Volumes for Docker, tar-ing files and folders, Postgres' safe backup opperations, and the `docker cp` command. Much more is to come of course. I haven't *actually* backed anything up yet, just started setting up the framework to do so. Stay tuned for more on this topic!

