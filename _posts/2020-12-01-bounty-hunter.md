---
layout: article
title:  "Bounty Hunter -  TryHackMe"
date:   2020-12-01 00:30:53 +0  530
# categories: [writeup, tryhackme]
# category: writeup
tags:
  - writeup
  - tryhackme
  - ctf
  - privesc
---

[Room Link](https://tryhackme.com/room/cowboyhacker)

# Task 1 : Living up to the title.

- Find open ports on the machine

![/assets/images/posts/bountyhunter/Untitled.png](/assets/images/posts/bountyhunter/Untitled.png)

![/assets/images/posts/bountyhunter/Untitled%201.png](/assets/images/posts/bountyhunter/Untitled%201.png)

- Who wrote the task list?

    `lin`

![/assets/images/posts/bountyhunter/Untitled%202.png](/assets/images/posts/bountyhunter/Untitled%202.png)

- What service can you bruteforce with the text file found?

    `ssh`

    Now do that bruteforce using `hydra`

- What is the users password?

    ![/assets/images/posts/bountyhunter/Untitled%203.png](/assets/images/posts/bountyhunter/Untitled%203.png)

Now login into `ssh` using the credentials obtained.

- user.txt

    ![/assets/images/posts/bountyhunter/Untitled%204.png](/assets/images/posts/bountyhunter/Untitled%204.png)

- root.txt

    Now escalating priviliges, I found that we had `sudo` permissions for `tar` so I found an exploit at `gtfobins`

    ![/assets/images/posts/bountyhunter/Untitled%205.png](/assets/images/posts/bountyhunter/Untitled%205.png)