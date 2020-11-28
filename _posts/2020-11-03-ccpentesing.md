---
layout: post
title:  "CC Pentesting - TryHackMe"
date:   2020-11-03 00:30:53 +0  530
categories: [writeup, tryhackme]
category: writeup
tags:
  - pentesting
  - sqli
  - privesc
---

Room : [https://tryhackme.com/room/ccpentesting](https://tryhackme.com/room/ccpentesting)

## Task 16 - [Section 5 - SQL Injection]: sqlmap

These questions can be answered from the help page `sqlmap -h`

1. How do you specify which url to check? `-u`
2. What about which google dork to use? `-g`
3. How do you select(lol) which parameter to use?(Example: in the url [http://ex.com?test=1](http://ex.com/?test=1) the parameter would be test.) `-p`
4. What flag sets which database is in the target host's backend?(Example: If the flag is set to mysql then sqlmap will only test mysql injections) `--dbms`
5. How do you select the level of depth sqlmap should use(Higher = more accurate and more tests in general)  `--level`
6. How do you dump the table entries of the database?  `--dump`
7. Which flag sets which db to enumerate? `-D`
8. Which flag sets which table to enumerate?  `-T`
9. Which flag sets which column to enumerate?   `-C`
10. How do you ask sqlmap to try to get an interactive os-shell?  `--os=shell`
11. What flag dumps all data from every table  `--dump-all`

## Task 17 - [Section 5 - SQL Injection]: A Note on Manual SQL Injection

`completed`

## Task 18 - [Section 5 - SQL Injection]: Vulnerable Web Application

1)  Set the url to the machine ip, and run the command 

`sqlmap -u http:\\<ip>`

![/assets/images/posts/ccpentest/Untitled.png](/assets/images/posts/ccpentest/Untitled.png)

It advises us to use `--forms` so we'll do that.

2) How many types of sqli is the site vulnerable too? `3`

`sqlmap -u [http://10.10.219.80](http://10.10.219.80/) --forms --batch`  

`--batch` assumes default values if any question is asked, when the command is running

![/assets/images/posts/ccpentest/Untitled%201.png](/assets/images/posts/ccpentest/Untitled%201.png)

From this command we find that this site is vulnerable to `3` types of sqli.

3) Dump the database. 

`sqlmap -u [http://10.10.219.80](http://10.10.219.80/) --forms --batch --dump`

![/assets/images/posts/ccpentest/Untitled%202.png](/assets/images/posts/ccpentest/Untitled%202.png)

This shows that the database fetched is `tests` and it has two tables `msg` and `lol`

4) What is the name of the database? `tests`

5) How many tables are in the database? `2`

`sqlmap -u [http://10.10.219.80](http://10.10.219.80/) --forms --batch -D tests --tables`

`-D` specifies the name of the database

`--tables` fetches the tables in that database

![/assets/images/posts/ccpentest/Untitled%203.png](/assets/images/posts/ccpentest/Untitled%203.png)

6) What is the value of the flag? `found_me`

we found it in the `--dump` command

![/assets/images/posts/ccpentest/Untitled%204.png](/assets/images/posts/ccpentest/Untitled%204.png)

## Task 19 - [Section 6 - Samba]: Intro

`got it`

## Task 20 - [Section 6 - Samba]: smbmap

For this task you can refer to the help page `smbmap --help`

1) How do you set the username to authenticate with? `-u`

2) What about the password? `-p`

3) How do you set the host? `-h`

4) What flag runs a command on the server(assuming you have permissions that is)? `-x`

5) How do you specify the share to enumerate? `-s`

6) How do you set which domain to enumerate? `-d`

7) What flag downloads a file? `--download`

8) What about uploading one? `--upload`

9) Given the username "admin", the password "password", and the ip "10.10.10.10", how would you run ipconfig on that machine

`smbmap -u "admin" -p "password" -H "10.10.10.10" -x "ipconfig"`

## Task 21 - [Section 6 - Samba]: smbclient

For this task you can refer to `smbclient --help` and [this](https://www.computerhope.com/unix/smbclien.htm)

1) How do you specify which domain(workgroup) to use when connecting to the host? `-W`

2) How do you specify the ip address of the host? `-I`

3) How do you run the command "ipconfig" on the target machine?  `-c "ipconfig"`

4) How do you specify the username to authenticate with? `-U`

5) How do you specify the password to authenticate with? `-P`

6) What flag is set to tell smbclient to not use a password? `-N`

7) While in the interactive prompt, how would you download the file test, assuming it was in the current directory? `get test`

8) In the interactive prompt, how would you upload your /etc/hosts file `put /etc/hosts`

## Task 22,23 - *No Answer Needed*

## Task 24 - [Section 7 - Final Exam]: Good Luck :D

Okay now we have to use what we learnt.

We are given the `ip` so let's scan it using `nmap` to see what services are running on its ports

![/assets/images/posts/ccpentest/Untitled%205.png](/assets/images/posts/ccpentest/Untitled%205.png)

We find that `Apache` server is running on `port 80`

Since we have a running web server we can  use `gobuster` to scan for hidden directories and files

![/assets/images/posts/ccpentest/Untitled%206.png](/assets/images/posts/ccpentest/Untitled%206.png)

`gobuster` scans may take time so be patient.

We see a `secret` directory is found. 

I found `secret.txt` file on scanning with `.txt` extension

![/assets/images/posts/ccpentest/Untitled%207.png](/assets/images/posts/ccpentest/Untitled%207.png)

The text file has some hash with a name which suggests its a username : password-hash pair

![/assets/images/posts/ccpentest/Untitled%208.png](/assets/images/posts/ccpentest/Untitled%208.png)

We can use `jtr` to decrypt the hash, and we find that the password and the username are the same.

![/assets/images/posts/ccpentest/Untitled%209.png](/assets/images/posts/ccpentest/Untitled%209.png)

![/assets/images/posts/ccpentest/Untitled%2010.png](/assets/images/posts/ccpentest/Untitled%2010.png)

Now that we have the credentials, we can login.

![/assets/images/posts/ccpentest/Untitled%2011.png](/assets/images/posts/ccpentest/Untitled%2011.png)

Once we are "in" , we can list the files.

There's a `user.txt` file, whose content answers our first question

![/assets/images/posts/ccpentest/Untitled%2012.png](/assets/images/posts/ccpentest/Untitled%2012.png)

Now we need to find `root.txt` . If you try to use `find` to search for it, you'll see that you don't have enough permissions to search through many directories. So we try to, what is known as, escalte our privilege.

`sudo -l` lists all the commands which are allowed to run as sudo

![/assets/images/posts/ccpentest/Untitled%2013.png](/assets/images/posts/ccpentest/Untitled%2013.png)

  

So, we run the found command using `sudo` to gain root access. Now that we use `find` command, we can easily locate the `root.txt` file and see its content.

![/assets/images/posts/ccpentest/Untitled%2014.png](/assets/images/posts/ccpentest/Untitled%2014.png)