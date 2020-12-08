---
layout: post
title:  "Introductory Researching - TryHackMe"
date:   2020-11-01 00:30:53 +0  530
categories: [writeup, tryhackme]
category: writeup
tags:
  - reconnaissance 
  - pentesting
---

[Room Link](https://tryhackme.com/room/introtoresearch)

# Task 1 : Introduction

nice stuff :)

# Task 2 : Example Research Question

- In the Burp Suite Program that ships with Kali Linux, what mode would you use to manually send a request (often repeating a captured request numerous times)?

    `repeater`

- What hash format are modern Windows login passwords stored in?

    `ntlm`

- What are automated tasks called in Linux?

    `cron jobs`

- What number base could you use as a shorthand for base 2 (binary)?

    `base 16`

- If a password hash starts with $6$, what format is it (Unix variant)?

    `SHA512crypt`

# Task 3 : Vulnerability Searching

- [ExploitDB](https://www.exploit-db.com/)
- [NVD](https://nvd.nist.gov/vuln/search)
- [CVE Mitre](https://cve.mitre.org/)

The below answers can be found on the ExploitDB website.

- What is the CVE for the 2020 Cross-Site Scripting (XSS) vulnerability found in WPForms?

    `CVE-2020-10385`

- There was a Local Privilege Escalation vulnerability found in the Debian version of Apache Tomcat, back in 2016. What's the CVE for this vulnerability?

    `CVE-2016-1240`

- What is the very first CVE found in the VLC media player?

    `CVE-2007-0017`

- If I wanted to exploit a 2020 buffer overflow in the sudo program, which CVE would I use?

    `CVE-2019-18634`

# Task 4 : Manual Pages

- SCP is a tool used to copy files from one computer to another. What switch would you use to copy an entire directory?

    `-r`  (`man scp | grep -i direct`)

- fdisk is a command used to view and alter the partitioning scheme used on your hard drive.

    What switch would you use to list the current partitions?

    `-l` (`man fdisk | grep -i list`)

- nano is an easy-to-use text editor for Linux. There are arguably better editors (Vim, being the obvious choice); however, nano is a great one to start with.

    What switch would you use to make a backup when opening a file with nano?

    `-B` (`man nano | grep -i backup`)

- Netcat is a basic tool used to manually send and receive network requests. What **command** would you use to start netcat in listen mode, using port 12345?

    `nc -l -p 12345` (usage : `man netcat`)

# Task 5 : Final Thoughts

n😇ice