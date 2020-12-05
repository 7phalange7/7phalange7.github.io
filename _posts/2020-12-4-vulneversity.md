---
layout: post
title:  "Vulnversity - TryHackMe"
date:   2020-12-04 00:30:53 +0  530
categories: [writeup, tryhackme]
category: writeup
tags:
  - pentesting
  - privesc
  - reconnaissance 
---

# Task 1 : Deploy the machine

deploy the machine using the green button and wait for the ip to load.

# Task 2 : Reconnaissance

run the nmap scan on the obtained ip address with the service versiob `-sV` flag

`nmap -sV target_ip`

this will take  2 - 3 minutes

![/assets/images/posts/vulneversity/Untitled.png](/assets/images/posts/vulneversity/Untitled.png)

> No exact OS matches for host (If you know what OS is running on it, see [https://nmap.org/submit/](https://nmap.org/submit/) ).

- Scan the box, how many ports are open ?

    `6`

- What version of the squid proxy is running on the machine?

    `3.5.12`

- How many ports will nmap scan if the flag -p-400 was used?

    `400`

- Using the nmap flag -n what will it not resolve?

    `dns`

- What is the most likely operating system this machine is running?

    `ubuntu` 

    I used `-O` flag but it said `No exact OS matches for host` so I used aggresive scan with `-T4` for faster output `sudo nmap -T4 -A <ip>`

    ![/assets/images/posts/vulneversity/Untitled%201.png](/assets/images/posts/vulneversity/Untitled%201.png)

- What port is the web server running on?

    `3333` 

    from the initial scan you can see that a `http` server is running on this port. You can also open the ip on this port on browser.

# Task 3 : Locating directories using GoBuster

Now scan the found web server with `gobuster` 

![/assets/images/posts/vulneversity/Untitled%202.png](/assets/images/posts/vulneversity/Untitled%202.png)

- What is the directory that has an upload form page?

    `/internal/` 

    ![/assets/images/posts/vulneversity/Untitled%203.png](/assets/images/posts/vulneversity/Untitled%203.png)

# Task 4 : Compromise the webserver

- Try upload a few file types to the server, what common extension seems to be blocked?

    `.php`

Now we will use `Burp Suite` to `fuzz` the upload form. What this means is that we will intercept the upload request and test different extenstions and find out which extension is allowed to upload, so that we can upload our `reverse shell` php script.

Open Burp Suite and [configure browser](https://portswigger.net/burp/documentation/desktop/getting-started/proxy-setup/browser).

Upload any file on browser and open `Proxy` tab 

![/assets/images/posts/vulneversity/Untitled%204.png](/assets/images/posts/vulneversity/Untitled%204.png)

Right click and send to inruder.

Now in the `Intruder` tab in `Positions` sub-tab `Clear` the default selection and `Add` selection by selecting the extension of your uploaded file.

![/assets/images/posts/vulneversity/Untitled%205.png](/assets/images/posts/vulneversity/Untitled%205.png)

![/assets/images/posts/vulneversity/Untitled%206.png](/assets/images/posts/vulneversity/Untitled%206.png)

Now in `Payloads` tab enter the extensions to test. and start attack in Positons tab.

![/assets/images/posts/vulneversity/Untitled%207.png](/assets/images/posts/vulneversity/Untitled%207.png)

![/assets/images/posts/vulneversity/Untitled%208.png](/assets/images/posts/vulneversity/Untitled%208.png)

We notice that the request for the `phtml` extension is of different length so we try to upload a phtml extension file and it is succesful

Now download the provide php script rename to `.phtml` and upload it through browser. 

Open a `netcat listener` on your machine and go to `http://<ip>:3333/internal/uploads/php-reverse-shell.phtml`

![/assets/images/posts/vulneversity/Untitled%209.png](/assets/images/posts/vulneversity/Untitled%209.png)

We are in!

- What is the name of the user who manages the webserver?

    `bill`

    go to the home directory, there you will find the user

- What is the user flag?

    you'll find the flag in `/home/bill/user.txt`

# Task 5 : Privilege Escalation

Since the task talks about `SUID` , lets find the executables with suid bit enabled

```bash
find / -perm -u=s -type f 2>/dev/null
```

- `/` specifes to find from root directory
- `perm -u=s` specifies the files with `SUID` bit enbled. We can also use `-4000` in place of `-u=s`
- `-type f` tells that we are looking for a regular file not a directory or special file.
- `2` denotes standard error and `/dev/null` is a special filesystem object that throws away everything written into it. So `2>/dev/null` is used to hide the errors by redirecting them to the null object.

![/assets/images/posts/vulneversity/Untitled%2010.png](/assets/images/posts/vulneversity/Untitled%2010.png)

- On the system, search for all SUID files. What file stands out?

    `bin/systemctl`

    this stands out , probably, because this is the only binary with `suid` exploit on `gtfobins`

[Gtfobins](https://gtfobins.github.io/gtfobins/systemctl/) tells us that `bin/systemctl` can be exploited if its suid bit is said.

![/assets/images/posts/vulneversity/Untitled%2011.png](/assets/images/posts/vulneversity/Untitled%2011.png)

So, we make minor adjustments as given in the description.

```bash
TF=$(mktemp).service
echo '[Service]
	Type=oneshot
	ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/flag"
	[Install]
	WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
cat /tmp/flag
```

We run the commands and get the flag

![/assets/images/posts/vulneversity/Untitled%2012.png](/assets/images/posts/vulneversity/Untitled%2012.png)