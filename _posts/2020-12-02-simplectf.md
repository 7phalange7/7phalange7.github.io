---
layout: article
title:  "Simple CTF - TryHackMe"
date:   2020-12-02 00:30:53 +0  530
# categories: [writeup, tryhackme]
# category: writeup
cover: assets/images/covers/simplectf.jpg
tags:
  - writeup
  - tryhackme
  - ctf
  - privesc
  - cms
---

[Room Link](https://tryhackme.com/room/easyctf)

- How many services are running under port 1000?

    `2` *the required ports are 80 and 21*

    ![/assets/images/posts/simplectf/Untitled.png](/assets/images/posts/simplectf/Untitled.png)

    For some reason nmap shows the port 80 closed, so the answer should be `1` but it turned out to be wrong. Therefore I performed an aggressive scan and found that port 80 service was also active.

    ![/assets/images/posts/simplectf/Untitled%201.png](/assets/images/posts/simplectf/Untitled%201.png)

- What is running on the higher port?

    `ssh`

Now, since we have a web-server at port 80 , we can use `gobuster` to find any hidden directories or files.

![/assets/images/posts/simplectf/Untitled%202.png](/assets/images/posts/simplectf/Untitled%202.png)

We find that there is a `CMS` website in the `/simple` directory

![/assets/images/posts/simplectf/Untitled%203.png](/assets/images/posts/simplectf/Untitled%203.png)

`CMS` or *Content Management Systems* are often vulnerable. We might be able to find a exploit for the particular CMS at Exploit-db

`CMS Made Simple` version 2.2.8

![/assets/images/posts/simplectf/Untitled%204.png](/assets/images/posts/simplectf/Untitled%204.png)

- What's the CVE you're using against the application?

    `CVE-2019-9053`

- To what kind of vulnerability is the application vulnerable?

    `sqli`

    Now we use the exploit found to actually exploit the cms website.

    I have python3 and python2 both installed , so I created a virtual environment to run the program and installed the required modules.

    Also this script is in python 2 as indicated by the interpreter.

    I ran these commands ( I saved the exploit as `cms.py`)

    The wordlist from the hint can be found at [this repo](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/best110.txt)

    ```bash
    pip install virtualenv
    virtualenv --version
    virtualenv cmshack
    virtualenv -p /usr/bin/python2.7 virtualenv_name
    source virtualenv_name/bin/activate
    pip install requests
    pip install termcolor

    python cmshack/cms.py -u http://<ip>/simple --crack -w best110.txt

    ```

    If everything goes right, you should get this. But for me and many others this just doesn't work.

    ![/assets/images/posts/simplectf/Untitled%205.png](/assets/images/posts/simplectf/Untitled%205.png)

- What's the password?

    `secret`

    If we try these credentials on ssh we are able to login

    ```bash
    ssh -p 2222 mitch@<ip>
    ```

- Where can you login with the details obtained?

    `ssh`

- What's the user flag?

    ![/assets/images/posts/simplectf/Untitled%206.png](/assets/images/posts/simplectf/Untitled%206.png)

- Is there any other user in the home directory? What's its name?

    `sunbath`

    ![/assets/images/posts/simplectf/Untitled%207.png](/assets/images/posts/simplectf/Untitled%207.png)

- What can you leverage to spawn a privileged shell?

    `sudo -l` lists all the binaries we are allowed to run as super user.

    We use that binary to spawn a root shell. check `gtfobins`

    ![/assets/images/posts/simplectf/Untitled%208.png](/assets/images/posts/simplectf/Untitled%208.png)

- What's the root flag?

    the flag is in `/root/root.txt`