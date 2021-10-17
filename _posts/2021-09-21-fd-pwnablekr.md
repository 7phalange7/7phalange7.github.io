---
layout: article
title:  "fd - pwnable.kr"
date:   2021-09-21 01:30:53 +0  530
tags:
  - writeup
  - pwnable.kr
  - file desctiptor
---

![Untitled](/assets/images/posts/fd//Untitled.png)

### Required Knowledge :

- what are file descriptors
- default file desciptors for stdin

`ssh [fd@pwnable.kr](mailto:fd@pwnable.kr) -p2222`

![Untitled](/assets/images/posts/fd//Untitled%201.png)

On carefully looking at the code, we can see that we need to *read* `LETMEWIN` string from the file desciptor `fd` which is determined by the argument of the binary.

So we can make fd = 0 to read from stdin and then we can enter the required string there.

![Untitled](/assets/images/posts/fd//Untitled%202.png)

Note : the argument here is 4660 i.e int(0x1234) becausue it is substracted in the code.