---
layout: article
title:  "collision - pwnable.kr"
date:   2021-09-21 02:30:53 +0  530
tags:
  - writeup
  - pwnable.kr
  - hash collision
---

> Daddy told me about cool MD5 hash collision today.
I wanna do something like that too! 
ssh col@pwnable.kr -p2222 (pw:guest)

apparently there is something called collision in hashes when two hashes become equal, read about it later.

This also has a flag , and an binary file which can access the flag file content. 

Fortunately , the source code has also been provided so we need not reverse engineer.

```cpp
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

On analysing the code, we can see that we need to pass in an argument of length `20 bytes`

Also when we sum every 4 bytes of it, it should give `0x21DD09EC`

So need four numbers of the given sum.
`0x6c5cec8 *4 + 0x6c5cecc` = 0x21DD09EC

so we can pass these numbers , but to pass them as bytes, we can use python

```cpp
./col `python -c "print '\xc8\xce\xc5\x06'*4 + '\xcc\xce\xc5\x06'"`
```

note : passed in *little endian* form

this will give the flag