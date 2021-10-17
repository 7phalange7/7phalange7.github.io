---
layout: article
title:  "random - pwnable.kr"
date:   2021-10-12 01:30:53 +0  530
tags:
  - writeup
  - pwnable.kr
  - rand
---

> Daddy, teach me how to use random value in programming!
ssh [random@pwnable.kr](mailto:random@pwnable.kr) -p2222 (pw:guest)
> 

three files on the ssh server : random,random.c,flag

downloading for better analysis:

```python
~/Downloads/pwnabl/random
❯ scp -P 2222 random@pwnable.kr:~/random.c ./
random@pwnable.kr's password: 
random.c                         100%  301     0.7KB/s   00:00    

~/Downloads/pwnabl/random 8s
❯ scp -P 2222 random@pwnable.kr:~/random ./  
random@pwnable.kr's password: 
random
```

`random.c` :

```c
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}
```

> If random numbers are generated with rand() without first calling srand(), your program will create the same sequence of numbers each time it runs.
> 

since no seed was provided, it will always generate the same no. which we can obtain by compiling and outputing the random no., and then we can pass the `if` condintion and cat the flag

```c
Last login: Tue Oct 12 13:53:50 2021 from 110.225.44.92
random@pwnable:~$ ./random
3039230856
Good!
Mommy, I thought libc random is unpredictable...
random@pwnable:~$
```