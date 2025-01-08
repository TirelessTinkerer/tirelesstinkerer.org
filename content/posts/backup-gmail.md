---
title: "Backup 18 years gmail and be able to access them everywhere"
date: 2024-01-05T21:36:50-08:00
draft: true
---

I've been using gmail for nearly 20 years. It records the history of my growth. It also spoiled me with amble amount of space so I seldom needed to delete and clean up my emails. Now I'm facing the problem of gmail space limit.

I want to backup my whole gmail and remove any email over 5 years old. However, I want to be able to still access all my old mails from anywhere. After some reseach, I decided to create a imap server through which I can access all my old (and new) emails. I don't plan to send emails from it. Here I want to record my process.

# Get existing gmails
To get a copy of all existing gmail, the easist way is to use google takeout.

A internet search also shows 2 tools: mbsync and offlineimap, both of which should be able to sync between gmail server to local copy. So techniquely, they can also be used to get all existing gmail to local. However, I never tried them because I heard that the oauth for gmail if painful.

Here is a good tutorial that I was able to follow to get mbsync working: https://frostyx.cz/posts/synchronize-your-2fa-gmail-with-mbsync#gmail-with-a-plain-password. The (tutorial from Arch)[https://wiki.archlinux.org/title/Isync#Configuring] is also good.

Of course, you can also use Thunderbird to download all gmail to local. There is another (Windows only) tool I found called MailStore, the Home version of which is free and can handle the gmail authentication easily. I tried both tools and they work well.

# Self host a mail server
Lots of web searches led me to dovecot.

`sudo apt install dovecot-imapd dovecot-pop3d
`


My next step will be to create docker container to have mbsync and dovecot setup and work together. Some reference I'll keep in mind:
1. https://www.reddit.com/r/selfhosted/comments/133avmq/dockerized_mbsync_dovecot_for_local_email_backup/
2. https://thehelpfulidiot.com/making-an-automatic-email-backup-updated-9-15-2024 This is what 1 is based on. It contains multiple parts.
