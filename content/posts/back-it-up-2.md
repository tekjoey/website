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

I ended up using Python, mostly because I feel more comfortable using it. I considered Bash, and may in the future convert the process to Bash if I dont end up using any Python-specific features.

At first I had one main monolithic script that interacted with every application. This was "easier" because it was a single file to edit, but it got very busy very fast. Eventually I want to do rotational backups (daily, weekly, monthly, etc), so thinking about coding all that logic into this main script became a bit overwhelming. I ended up breaking it up so one backup script handles one applications requirements. Each of these individual backup scripts will be called one by one from a main script. Currently that main script is just a bash script that calls each Python file like so:

```bash
#!/bin/bash
/docker/authentik/backup.py
/docker/calibre-web/backup.py
/docker/collabora-code/backup.py
.
.
.
```

This allows for flexability with each application's backup needs. It means that each applications's backup needs are self-contained and not intertwined with any other application. Hopefully, this will scale well and be more maintainable. Only time will tell.

Within each of these backup scripts I have logic for backing up the database (if applicable) and encrypting the `.env` files. Since these tasks are standard accross the containers and the logic is the same, I created a module that contains all this logic that is simply imported in each script. I couldnt find out exactly how to do a clean import, so I end up modifying the Python PATH first with this code:

```python
#/docker/authentik/backup.py
import sys
sys.path.append('/docker/backup')
import backup_utils as bu
```

Then in `/docker/backup` I have a file called `backup_utils.py` that contains all the common functions. I will eventually push all my docker config files to my [GitHub](https://github.com/tekjoey), so you can view them there.

It was also at this time that I started using git. I had used it before, mostly through VSCode, never really through the command line. I won't go into detail here since all I did was basics like `git init`, `git status`, `git branch`, `git checkout {branch}`, `git add {file}`, and of course `git commit -m {messsage}`.

When I innitially set about doing this project I thought I could create TrueNAS snapshots via API. While that is true (sorta, they use websockets which is a whole different thing tha I was expecting), these manually created snapshots are not part of any automated lifecycle tasks and so would have to be manually deleted (wither through the GUI or API). I decided that was too much complexity, so instead I will simply schedule the script to run before the snapshots.
