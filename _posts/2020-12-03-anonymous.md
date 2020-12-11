---
layout: post
title:  "Anonymous - TryHackMe"
date:   2020-12-03 00:30:53 +0  530
categories: [writeup, tryhackme]
category: writeup
tags:
  - ctf
  - privesc
  - ftp
  - smb
---

[Room Link](https://tryhackme.com/room/anonymous)

# Task 1 : Pwn

- Enumerate the machine. How many ports are open?

    `4`

    ![/assets/images/posts/anonymousthm/Untitled.png](/assets/images/posts/anonymousthm/Untitled.png)

    Normal scan shows only 2 ports open but it was wrong. So I scanned all the ports and it took 20 minutes.

    ![/assets/images/posts/anonymousthm/Untitled%201.png](/assets/images/posts/anonymousthm/Untitled%201.png)

- What service is running on port 21?

    `ftp`

- What service is running on ports 139 and 445?

    `smb`

- There's a share on the user's computer. What's it called?

    `pics`

    you can enumerate the smb shares using `enum4linux -S <ip>`

    ![/assets/images/posts/anonymousthm/Untitled%202.png](/assets/images/posts/anonymousthm/Untitled%202.png)

Let's try to login into the `ftp` server. 

![/assets/images/posts/anonymousthm/Untitled%203.png](/assets/images/posts/anonymousthm/Untitled%203.png)

you can type `help` to see the available commands.

There is a directory and it has a script `[clean.sh](http://clean.sh)` 

Download all three files using `get` to view content.

![/assets/images/posts/anonymousthm/Untitled%204.png](/assets/images/posts/anonymousthm/Untitled%204.png)

looking at the `log` file it seems that the script runs repeatedly after an interval.

So we upload our own code into the script and upload it to ftp.

![/assets/images/posts/anonymousthm/Untitled%205.png](/assets/images/posts/anonymousthm/Untitled%205.png)

remember to add your own ip here

Now open a netcat listener, and in a minute you recieve a bash shell.

![/assets/images/posts/anonymousthm/Untitled%206.png](/assets/images/posts/anonymousthm/Untitled%206.png)

get the user flag

![/assets/images/posts/anonymousthm/Untitled%207.png](/assets/images/posts/anonymousthm/Untitled%207.png)

Now we need to escalate privileges. We list all SUID binaries

```bash
find / -perm -u=s -type f 2>/dev/null
```

We can find an SUID exploit for `env` at `gtfobins`

![/assets/images/posts/anonymousthm/Untitled%208.png](/assets/images/posts/anonymousthm/Untitled%208.png)

Now get the root flag

![/assets/images/posts/anonymousthm/Untitled%209.png](/assets/images/posts/anonymousthm/Untitled%209.png)