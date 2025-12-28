---
title: 'Back It Up - Part 2'
date: '2025-12-10'
tags:
    - backups
categories:
    - Homelab
draft: true
# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
description: "Part two in my ongoing backup journey"
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

Welp, I'm ready to start writing my backup script!

I begin with the databases. Only a few of my containers have seperate databse containers. Those are Authentik, Nextcloud, Paperless-NGX, and Immich. The first thing to do is to get the name of each container. At this point I realized that my naming schene is a bit inconsistant. Paperless, for example has `container_name` keys for each of the containers in teh stack, while the other three don't, so they default to Compose's default, which is `project-service-index`. Project is the folder name the compose file is in, and service is the key directly under `services:` in the yaml file that defines the actual service. So my Nextcloud file, for example, defines two services `app` and `db`. The compose file is in the folder of `nextcloud` so the container names are `nextcloud-app-1` and `nextcloud-db-1` respectively. This works great, and at first i was thinking to rename the paperless containers to follow this convention, but for reasons discussed later, i think i might actually go back to manually naming each container.
So anyway, I got all the database container names and loaded them into a bash script file.

```bash
#!/bin/bash

#Dump all databases
db_containers=("authentik-postgresql-1" "immich_postgres" "nextcloud-db-1" "paperless_db")
```

Then i go to write the code to actually dump the databases 

```bash
for db in ${db_containers[@]}; do
        docker exec $db psql -U ...
```

I need a user for each databse. They all use different users. If i was writing this in Python, i could use dictionaries...does Bash support dictionaries....?

Indeed it does! They are caled associative arrays, and they allow for key:value pairings. So I go to update my array, now with the user associated with each database.

```bash
db_containers=(["authentik-postgresql-1"]="authentik" ["immich_postgres"]="postgres" ["nextcloud-db-1"]="nextcloud" ["paperless_db"]="paperless")
```

Then to reformat my for loop... wait, the `pg_dump` command also needs to specify a database to dump. yet again, they are all different, and not necessarily based on the name. Bash DOESN'T support nested arrays. Well, will Python work?

I looked into using Python. The main thing that made me uncomfortable was there wasnt a solid way to run shell commands. There are certainly ways, but i felt more comfortable using straight bash, so I quickly abandoned Python.

So i looked more into PostgreSQL data dumping and found `ps_dumpall` which dumps all databases on a server, so all you need is the user. Works for me! `pg_dumpall` also writes to `stdout` which makes it easier because I can capture that and write to a file on the Docker host, instead of having to copy files out of the container.

So, after adding some datetime information to the file name, my backup script looks like this:

```bash
#!/bin/bash

## Dump All Databases
declare -A db_containers=(["authentik-postgresql-1"]="authentik" ["immich_postgres"]="postgres" ["nextcloud-db-1"]="nextcloud" ["paperless_db"]="paperless")

for db in ${!db_containers[@]}; do
        username="${db_containers[$db]}"
        docker exec $db pg_dumpall -U $username > /docker/backup/$db-$(date +"%Y-%m-%d_%H-%M").sql
        echo $db has been dumped
done
```







for db in ${!db_containers[@]}; do
        username="${db_containers[$db]}"

        docker exec $db pg_dumpall -U $username > /docker/backup/$db-$(date +"%Y-%m-%d_%H-%M").sql
        echo $db has been dumped


db_containers=(["authentik-postgresql-1"]="authentik" ["immich_postgres"]="postgres" ["nextcloud-db-1"]="nextcloud" ["paperless_db"]="paperless")
