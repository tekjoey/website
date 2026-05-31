---
title: 'The Database Was From the Future'
date: '2026-05-31'
tags:
    - homelab
    - mistakes
    - backups
categories:
    - Homelab
draft: true

description: or 'This is why we have backups, people!'

---

The homelab is intended to be a place to make mistakes, and this is good because I make my fair share number of them. Today's mistakes involved inconsistant databses, ChatGPT, and (thankfully!) usefull backups to save the day! I'll walk you through the incident the same way I experienced it.

I began as I was working to deploy a new service (yes, ANOTHER new one), this time a bookmarking app called Karakeep. It has OIDC suport, so I went to authentik to setup the connection. 
I was intially met with the normal login screen, but after I typed in my username and password I was then met with a generic "Server Error" message. 
"That's odd" I muttered as i tried logging in through a private window and with a different account. All three attemts resulting in the same "serer Error" message.

So, I checked the logs and recieved this...

[[Insert log file screenshot]]

I had...no idea what I was looking at and why anything like this would have come up. So I threw it into ChatGPT and asked it what the heck is going on here. 
ChatGPT respoded by pointing to the first line (`relation: "authentik_core_user_ak_groups" does not exist`) explaing that a 'relation' in PostgreSQL is basically a table. Authentik is trying to query the table `authentik_core_user_ak_groups` and PostgreSQL is responding saying it dosn't exist.
This results in a fatal error because that table *should* exist. ChatGPT hypothosized that this was due to a failed or incomplete upgrade where some databse migrations didn't happen properly.

Based on this diagnosis, I knew that I was well out of my depth. Though I know some things, I am not a Database Administrator, so I decied to let ChatGPT guide me through the most likely fixes. 
I know enough about enough that I could spot wheather a command that ChatGPT was going to ask me to run would 'accidentally' delete all my data.

It was at this point that I rememebred that I had started upgrading Authentik about a week or so ago. I was a few versions behind and had a few minutes, but for some reason I only updated by one version. Maybe something went wrong durring that upgrade?

ChatGPT had me check various things trying to see if the databse migrations failed or if some wern't applied. Instead of rehahsing it all, here is a link to my conversation so you can see in real time all the stuff that was tried

https://chatgpt.com/share/6a1c7f33-5b38-83ea-8e0b-78a3ca6b5465

Near the end we determined that all the migrations had happened successfully and completely. There were no inconsistantcies in the databse or in authentik's code. 
Each were saying that everything was applied, but clearly they had different ideas about what "everything" means.
Since that was the case the only option left was to roll back to an older version and restore from a backup. I went digging in my github repo with all my encypted envronment files.
To my surprise, the version that was curently installed and running (2025.8) was the same version that had been running for the past 3 months. But I had upgraded, haden't I?

It was at this point that my brain decided to finally give me all the information from that fateful day. I had indeed upgraded to 2025.10. The database migrations were comeplete and successfull. 
Then, I decided I didn't have enough time to upgrade all the way to the newest version (2026.something), so I just changed the version tag back to 2025.8 and re-deployed, leaving the database with all the changed made in the 2025.10 upgrade.

So, the 2025.8 code was looking for a table `authentik_core_user_ak_groups` that had been removed in 2025.10. This what was causing the inconsistancy. There was no failure, just two incompatable systems.

Thankfully, this happened only a week ago. I export all the databases for my apps every night and keep 30 days worth of these backups. The restore process was as simple as stopping all but the db container, removing the current databse, and restoring the backup to a new databse.
Then I started up all the containers, and voila, I was able to login no problem and finally create the new provider for Karakeep!

So, lessons learned? Don't rollback upgrades without actually rolling them back completely. Oh and also, if an upgrade is successfull, leave it. Sounds obvious, but hey, we all have moments of stupidity sometimes. 

(I still have no reason why I didnt just leave it at 2025.10)
