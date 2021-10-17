---
layout: article
title:  "Agent Sudo - TryHackMe"
date:   2020-12-06 00:30:53 +0  530
# categories: [writeup, tryhackme]
# category: writeup
tags:
  - writeup
  - tryhackme
  - ctf
  - privesc
  - bruteforce
  - steg
---

[Room Link](https://tryhackme.com/room/agentsudoctf)

# Task 1 : Author note

lets do it..

# Task 2 : Enumerate

let us start with an nmap scan

![/assets/images/posts/agentsudo/Untitled.png](/assets/images/posts/agentsudo/Untitled.png)

### How many open ports?

`3`

Now let us scan the http sever with `gobuster`

### How you redirect yourself to a secret page?

`user-agent`

lets open the http port on web browser

![/assets/images/posts/agentsudo/Untitled%201.png](/assets/images/posts/agentsudo/Untitled%201.png)

### What is the agent name?

a user-agent is a string in your request that identifies your browser, example "chrome on android". This is used by websites to serve webpages depending upon the browser and device.

The hint want us to use the user-agent `C` , so we will change the user-agent in the request using `Burp Suite.`

Once you reload the page making the request as `C` you get the following page, which tells the agent name

![/assets/images/posts/agentsudo/Untitled%202.png](/assets/images/posts/agentsudo/Untitled%202.png)

# Task 3 : Hash cracking and brute-force

now comes the interesting part. We have got the username and we know that the password is weak. So we can bruteforce it.

### FTP password

I'll brute force ftp with `hydra` and we get a password

![/assets/images/posts/agentsudo/Untitled%203.png](/assets/images/posts/agentsudo/Untitled%203.png)

### Zip file password

let us login into ftp

![/assets/images/posts/agentsudo/Untitled%204.png](/assets/images/posts/agentsudo/Untitled%204.png)

we can download these files on our system using `get` command

![/assets/images/posts/agentsudo/Untitled%205.png](/assets/images/posts/agentsudo/Untitled%205.png)

It looks like we have to find hidden password in the images.

### steg password

after tinkering around a bit I found that `cutie.png` has hidden files in it. These can be extracted with `binwalk`

![/assets/images/posts/agentsudo/Untitled%206.png](/assets/images/posts/agentsudo/Untitled%206.png)

we have a zip and a text file which is empty

![/assets/images/posts/agentsudo/Untitled%207.png](/assets/images/posts/agentsudo/Untitled%207.png)

When we try to extract the zip, it demands a password. So we will find the password using `zip2john` and `john the ripper` which is indicated by the hint `Mr.John`

![/assets/images/posts/agentsudo/Untitled%208.png](/assets/images/posts/agentsudo/Untitled%208.png)

Now we can extract the zip using the password obtained, looks like it contained the same text file, but now it has some content

![/assets/images/posts/agentsudo/Untitled%209.png](/assets/images/posts/agentsudo/Untitled%209.png)

We have obtained a phrase but it looks encoded (not really tho, I tried it and it does not work so I thought it must be encoded). Since we can't determine what encoding is it, let's use `magic` recipie of `cyberchef`

![/assets/images/posts/agentsudo/Untitled%2010.png](/assets/images/posts/agentsudo/Untitled%2010.png)

Now this looks like an usable phrase.

Let us look at the other image. Using `steghide` to extract, it demands a passphrase and luckily we just obtained it.

![/assets/images/posts/agentsudo/Untitled%2011.png](/assets/images/posts/agentsudo/Untitled%2011.png)

so we obtained the full name and ssh password

### Who is the other agent (in full name)?

obtained in the prev step

### SSH password

obtained in the prev step

!!! THM will accept the password without `!` but while logging in make sure to inlcude `!`

# Task 4 : Capture the user flag

Now let us ssh using the pass obtained

### What is the user flag?

![/assets/images/posts/agentsudo/Untitled%2012.png](/assets/images/posts/agentsudo/Untitled%2012.png)

### What is the incident of the photo called?

`roswell alien autopsy`

you can copy the image on your system using `scp`

![/assets/images/posts/agentsudo/Untitled%2013.png](/assets/images/posts/agentsudo/Untitled%2013.png)

after a little googling and reverse image search, you can find the answer

# Task 5 : Privilege escalation

lets check which binaries can be run as sudo.

![/assets/images/posts/agentsudo/Untitled%2014.png](/assets/images/posts/agentsudo/Untitled%2014.png)

we find that we can run `/bin/bash` as any user except root

Let us do a google search about it 

![/assets/images/posts/agentsudo/Untitled%2015.png](/assets/images/posts/agentsudo/Untitled%2015.png)

we find there is related exploit and we can confirm version compatibility using `sudo -V`

### CVE number for the escalation

`CVE-2019-14287`

![/assets/images/posts/agentsudo/Untitled%2016.png](/assets/images/posts/agentsudo/Untitled%2016.png)

### What is the root flag?

So we find out from that we can gain access by running `sudo -u#-1 /bin/bash`

This version of sudo interprets this as `sudo -u 0 /bin/bash` indrectly ( 0 is the root user ) . Since we didn't directly use 0 sudo runs it with the given user which effectively is root 

![/assets/images/posts/agentsudo/Untitled%2017.png](/assets/images/posts/agentsudo/Untitled%2017.png)

### (Bonus) Who is Agent R?

`Deskel`