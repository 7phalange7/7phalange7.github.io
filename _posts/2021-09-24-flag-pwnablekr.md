---
layout: article
title:  "flag - pwnable.kr"
date:   2021-09-24 01:30:53 +0  530
tags:
  - writeup
  - pwnable.kr
  - gdb catchpoint
  - packed binary
  - upx
---

> Papa brought me a packed present! let's open it.
Download : [http://pwnable.kr/bin/flag](http://pwnable.kr/bin/flag)
This is reversing task. all you need is binary
> 

So in this challenge we are given a binary, no src code.

So, first the file command

```bash
❯ file flag
flag: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
```

hmmm, very less info here.

On running strings or examining in ghidra I found this :

```bash
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
```

So I searched a bit about packers and UPX,

> upx - compress or expand executable files
> 

So, upx packs (compresses) the binary file and decompreses at runtime, so you won't find anything in gdb or ghidra.

So we can either unpack it using upx or stop the execution during runtime, dump the memory and examine it.

## Method 1 : unpack using upx

```bash
upx -d flag
```

now running file on the unpacked binary

```bash
❯ file flag
flag: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 2.6.24, BuildID[sha1]=96ec4cc272aeb383bd9ed26c0d4ac0eb5db41b16, not stripped
```

aha!

Now use gdb or strings to get the flag

![Untitled](/assets/images/posts/flag//Untitled.png)

## Method 2: generate core file

Examining the orignial binary with gdb won't give any results.

So we need to stop the binary during execution and examine it.

Using `strace` we can see the system calls made by the program

![Untitled](/assets/images/posts/flag//Untitled%201.png)

Here a bunch of syscalls have been made, we can examine them but the main thing to note is that when `exit_group()` syscall is made , the malloc and strcpy would have already happended and the string would be in memory somewhere.

[GDB catchpoints](https://undo.io/resources/gdb-watchpoint/gdb-catchpoints/)

So catching the syscall using gdb

```bash
gdb-peda$ catch syscall exit_group
Catchpoint 1 (syscall 'exit_group' [231])
```

now running the binary, it will stop at the catchpoint and then we can dump the core file( dump the state of the program at that instance)

```bash
gdb-peda$ generate-core-file flag-core-file
```

now we can use strings on this to get the flag :)

`UPX...? sounds like a delivery service :)`