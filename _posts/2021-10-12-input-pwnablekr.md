---
layout: article
title:  "input - pwnable.kr"
date:   2021-10-15 01:30:53 +0  530
tags:
  - writeup
  - pwnable.kr
  - environment variables
  - file descriptor
  - socket programming
  - unix processes
---


> Mom? how can I pass my input to a computer program?
ssh [input2@pwnable.kr](mailto:input2@pwnable.kr) -p2222 (pw:guest)
> 

so here also we have three files

```c
input2@pwnable:~$ ls -l
total 24
-r--r----- 1 input2_pwn root      55 Jun 30  2014 flag
-r-sr-x--- 1 input2_pwn input2 13250 Jun 30  2014 input
-rw-r--r-- 1 root       root    1754 Jun 30  2014 input.c
```

**`input.c`**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
	printf("Welcome to pwnable.kr\n");
	printf("Let's see if you know how to give input to program\n");
	printf("Just give me correct inputs then you will get the flag :)\n");

	// argv
	if(argc != 100) return 0;
	if(strcmp(argv['A'],"\x00")) return 0;
	if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
	printf("Stage 1 clear!\n");	

	// stdio
	char buf[4];
	read(0, buf, 4);
	if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
	read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
	printf("Stage 2 clear!\n");
	
	// env
	if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
	printf("Stage 3 clear!\n");

	// file
	FILE* fp = fopen("\x0a", "r");
	if(!fp) return 0;
	if( fread(buf, 4, 1, fp)!=1 ) return 0;
	if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
	fclose(fp);
	printf("Stage 4 clear!\n");	

	// network
	int sd, cd;
	struct sockaddr_in saddr, caddr;
	sd = socket(AF_INET, SOCK_STREAM, 0);
	if(sd == -1){
		printf("socket error, tell admin\n");
		return 0;
	}
	saddr.sin_family = AF_INET;
	saddr.sin_addr.s_addr = INADDR_ANY;
	saddr.sin_port = htons( atoi(argv['C']) );
	if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("bind error, use another port\n");
    		return 1;
	}
	listen(sd, 1);
	int c = sizeof(struct sockaddr_in);
	cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
	if(cd < 0){
		printf("accept error, tell admin\n");
		return 0;
	}
	if( recv(cd, buf, 4, 0) != 4 ) return 0;
	if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
	printf("Stage 5 clear!\n");

	// here's your flag
	system("/bin/cat flag");	
	return 0;
}
```

This was a tough one, not that tough, but since it involved quite a lot of new topics to learn it took me some time.

It involves a lot of c code that can be pwn-ed using a c script more easily than writing a python script. So the script for this challenge will be in `c`.

I'll solve the stages one by one and keep on building upon the script. 

### Stage 1 : argv

```c
	// argv
	if(argc != 100) return 0;
	if(strcmp(argv['A'],"\x00")) return 0;
	if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
	printf("Stage 1 clear!\n");	
```

`argc` is the number of arguments passed to the program (including the program name)

`argv` is the string array that contains all the arguments

So to clear this we need to ensure three things :

1. number of arguments is 100 (so pass 99 arguments after binary name)
2. 'A'th i.e 65th argument should be `\x00`
3. 66th argument should be `\x20\x0a\x0d`

The hex escape sequence have a lot of special characters (like NULL, \n, SPACE.... see `man ascii`) which makes it difficult to pass it directly in the command line, so we will pass it through our `c` code script.

**Note : an important thing I found was that out argument list should end with `NULL` if we are passing through a script.** In command line it must automatically add a NULL char as in case of string stdin

So we have to pass a total of `101` arguments.

We can use `exec` system call in c to run our binary.

It has a bunch of variations, I'll use `execve` which allows to pass environment variables too(stage 3).

See `man execve` for detailed info.

[Exec System Call in C](https://linuxhint.com/exec_linux_system_call_c/)

So for our code for stage 1 :

```c
#include<unistd.h> //execve
int main()
{
	char *arg[101] = {[0 ... 99] = "a"}; // nicee
	arg['A'] = "\x00";
	arg['B'] = "\x20\x0a\x0d";
	arg[100] = NULL;

	execve("./input",arg,NULL);
	return 0;
}
```

***Stage 1 clear!***

### Stage 2: stdio

```c
// stdio
	char buf[4];
	read(0, buf, 4);
	if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
	read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
	printf("Stage 2 clear!\n");
```

This took me the longest as I had to learn a lot of new concepts.

`memcmp` is similar to strcmp

> `strcmp` compares null-terminated C strings
`strncmp` compares at most N characters of null-terminated C strings
`memcmp` compares binary byte buffers of N bytes
> 

read() def:

`ssize_t read(int fd, void *buf, size_t count);`

it takes a file desctiptor fd , and reads count bytes from that file into buf

here once it is reading from `stdin` 0

and once from `stderr` 2

We need to find a way to pass the required buffer to stdin and stderr to be read by this binary.

So apparently we can do this by using `pipe` in c. In layman terms, we will create another process , 

write from that process to a pipe , then map that pipe to stdin and then let our binary read from stdin.

So for this some theory :

*learn about Linux process, fork(), pipe(),dup()*

**Playlist : UNIX processes in C**

[The fork() function in C](https://www.youtube.com/watch?v=cex9XrZCU14&list=PLfqABt5AS4FkW5mOn2Tn9ZZLLDwA3kZUY)

[fork() in C - GeeksforGeeks](https://www.geeksforgeeks.org/fork-system-call/)

[pipe() System call - GeeksforGeeks](https://www.geeksforgeeks.org/pipe-system-call/)

[dup() and dup2() Linux system call - GeeksforGeeks](https://www.geeksforgeeks.org/dup-dup2-linux-system-call/)

> A tricky use of dup2() system call: As in dup2(), in place of newfd any file descriptor can be put. Below is a C implementation in which the file descriptor of Standard output (stdout) is used. This will lead all the printf() statements to be written in the file referred by the old file descriptor.
> 

[http://unixwiz.net/techtips/remap-pipe-fds.html](http://unixwiz.net/techtips/remap-pipe-fds.html)

So code untill stage 2:

```c
#include <stdio.h>
#include<unistd.h>
int main()
{
	char *arg[101] = {[0 ... 99] = "a"};
	arg['A'] = "\x00";
	arg['B'] = "\x20\x0a\x0d";
	arg[100] = NULL;

	int pipetostdin[2],pipetostderr[2]; 

	if(pipe(pipetostdin)<0 || pipe(pipetostderr)<0) // remember to create pipe before fork()
		{
			printf("failed to create pipe\n");
		}

	pid_t childpid = fork(); //pid_t is integer

	if(childpid < 0 )
	{
		printf("failed to fork\n");
	}

	
	if(childpid == 0)
	{
		//inside child process

		close(pipetostderr[0]); //close read pipes
		close(pipetostdin[0]);

		write(pipetostdin[1],"\x00\x0a\x00\xff", 4);
		write(pipetostderr[1],"\x00\x0a\x02\xff", 4);

		close(pipetostdin[1]);	// close write pipes
		close(pipetostderr[1]);
	}
	else
	{
		//inside parent process

		close(pipetostdin[1]);	// close write pipes
		close(pipetostderr[1]);

		dup2(pipetostdin[0],0); //map read pipes to stdin and stderr
		dup2(pipetostderr[0],2);

		close(pipetostderr[0]); //close read pipes
		close(pipetostdin[0]);

		execve("./input",arg,NULL);

	}
	
	return 0;
}
```

*explanation* : pipes can be used to communicate between two processes but here we are modifying this use case to fit our requirements.

We create two pipes namely pipetostdin, pipetostderr. Then in the child process we write to these pipes from the write end.

In the parent process we read from these pipe's read end (when execve will execute our binary) but before that we map the read ends to stdin and stderr, so that pipe can be read using stdin and stderr.

Remember to close the unused end(read/write).

***Stage 2 clear!***

### Stage 3: env

```c
// env
	if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
	printf("Stage 3 clear!\n");
```

This one was quite easier.

`getenv` : it returns the environment variable pointed to by the argument string.

So we need to pass an environment whose name is `\xde\xad\xbe\xef` and value is `\xca\xfe\xba\xbe` .

> What is environment variable in C?
> 

> Environment variable is a variable that will be available for all C applications and C programs. Once environment variables are exported, we can access them from anywhere in a C program without declaring and initializing in an application or C program.
> 

This might give us a clearer picture :

```c
ALLUSERSPROFILE=C:\ProgramData
CommonProgramFiles=C:\Program Files\Common Files
HOMEDRIVE=C:
NUMBER_OF_PROCESSORS=2
OS=Windows_NT
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
PROCESSOR_ARCHITECTURE=x86
PROCESSOR_IDENTIFIER=x86 Family 6 Model 42 Stepping 7, GenuineIntel
PROCESSOR_LEVEL=6
PROCESSOR_REVISION=2a07
ProgramData=C:\ProgramData
ProgramFiles=C:\Program Files
PUBLIC=C:\Users\Public
SESSIONNAME=Console
SystemDrive=C:
SystemRoot=C:\Windows
WATCOM=C:\watcom
windir=C:\Windows
```

> An alternative method of accessing the environment list is to declare a third argument to the main() function:
`int main(int argc, char *argv[], char *envp[])`
This argument can then be treated in the same way as environ, with the difference that its scope is local to main(). Although this feature is widely implemented on UNIX systems, its use should be avoided since, in addition to the scope limitation, it is not specified in SUSv3.
But most of the compilers also support a third declaration of main that accepts third argument. The third argument stores all environment variables.
> 

remember we used `execve` in first stage to exeute our binary, we can use that to pass a custom evnvironment varibale.

we can do that as :

```c
char *envar[2] = {"\xde\xad\xbe\xef=\xca\xfe\xba\xbe",NULL};  //remember to add NULL

execve("./input",arg,envar);
```

so just need to add this one line.

***Stage 3 clear!***

### Stage 4: file

```c
	// file
	FILE* fp = fopen("\x0a", "r");
	if(!fp) return 0;
	if( fread(buf, 4, 1, fp)!=1 ) return 0;
	if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
	fclose(fp);
	printf("Stage 4 clear!\n");
```

This one was easy too.

Basically what's happening here?

a file `\x0a` is opened in read mode, 1 item of 4 byte into buf and then comapares it to a string.

```c
size_t fread(void *ptr, size_t size, size_t n, FILE *stream);

size_t fwrite(const void *ptr, size_t size, size_t n, FILE *stream);
```

> The  function  `fread()` reads `n` items of data, each `size` bytes long, from the stream pointed to by `stream`, storing them at the location given by `ptr`.
> 

So, we just need to reverse it - write in a file of same name, the data which we want

code for stage 4:

```c
FILE *fp = fopen("\x0a","w");      
	fwrite("\x00\x00\x00\x00",4,1,fp);
	fclose(fp);
```

```c
#include <stdio.h>
#include<unistd.h>

int main()
{
	char *arg[101] = {[0 ... 99] = "a"};
	arg['A'] = "\x00";
	arg['B'] = "\x20\x0a\x0d";
	arg[100] = NULL;

	char *envar[2] = {"\xde\xad\xbe\xef=\xca\xfe\xba\xbe",NULL};

	FILE *fp = fopen("\x0a","w");      
	fwrite("\x00\x00\x00\x00",4,1,fp);
	fclose(fp);

	int pipetostdin[2],pipetostderr[2];

	if(pipe(pipetostdin)<0 || pipe(pipetostderr)<0)
		{
			printf("failed to create pipe\n");
		}

	pid_t childpid = fork();

	if(childpid < 0 )
	{
		printf("failed to fork\n");
	}

	
	if(childpid == 0)
	{
		//inside child process

		close(pipetostderr[0]); //close read pipes
		close(pipetostdin[0]);

		write(pipetostdin[1],"\x00\x0a\x00\xff", 4);
		write(pipetostderr[1],"\x00\x0a\x02\xff", 4);

		close(pipetostdin[1]);	// close write pipes
		close(pipetostderr[1]);
	}
	else
	{
		//inside parent process

		close(pipetostdin[1]);	// close write pipes
		close(pipetostderr[1]);

		dup2(pipetostdin[0],0); //map read pipes to stdin and stderr
		dup2(pipetostderr[0],2);

		close(pipetostderr[0]); //close read pipes
		close(pipetostdin[0]);

		

		execve("./input",arg,envar);

	}

	
	return 0;
}
```

***Stage 4 clear!***

### Stage 5: network

```c
// network
	int sd, cd;
	struct sockaddr_in saddr, caddr;
	sd = socket(AF_INET, SOCK_STREAM, 0);
	if(sd == -1){
		printf("socket error, tell admin\n");
		return 0;
	}
	saddr.sin_family = AF_INET;
	saddr.sin_addr.s_addr = INADDR_ANY;
	saddr.sin_port = htons( atoi(argv['C']) );
	if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("bind error, use another port\n");
    		return 1;
	}
	listen(sd, 1);
	int c = sizeof(struct sockaddr_in);
	cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
	if(cd < 0){
		printf("accept error, tell admin\n");
		return 0;
	}
	if( recv(cd, buf, 4, 0) != 4 ) return 0;
	if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
	printf("Stage 5 clear!\n");
```

This one looks the scariest, and justly so. But it is doable if you don't want to go into details of networking and socket programming in c.

I too don't have a thorough knowledge of these concepts yet, but I got enough to solve this.

Basically sockets are nodes between which communication happens.

One will be server and several clients can try an communicate with it.

To get a better idea of this I referred to these resources :

[Networking and Socket Programming Tutorial in C](https://www.codeproject.com/articles/586000/networking-and-socket-programming-tutorial-in-c)

[networking](https://www.youtube.com/playlist?list=PL6kv90IDxefUDFFGcD6w3-0H1muF1I8_X)

So what is happening here is, that a server is setup and it listens for a connection from a client and then receives some data from the client, we need to control that data somehow.

For this we will setup a client and connect and send the required data.

For this we need the server and client running in different processes and also need to make sure the client does not send data before server is up.

We already have two processes child and parent ( when execve is called, it will terminate the parent processes and start a new processes and server will run in this processes.) 

We can use the child process to run the client. Also we will make it sleep for 5 seconds so that server gets enough time to get setup-ed.

code for stage 5:

```c
// in child processes
		sleep(5);

		int sd;
		struct sockaddr_in saddr;
		sd = socket(AF_INET,SOCK_STREAM,0);
		if(sd == -1){
		printf("socket error\n");
		return 0;
		}
		saddr.sin_family = AF_INET;
		saddr.sin_addr.s_addr = inet_addr("127.0.0.1");
		saddr.sin_port = htons( 5000);
		if(connect(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("connect error, use another port\n");
    		return 1;
		}

		write(sd,"\xde\xad\xbe\xef", 4);

		close(sd);
```

here we are connecting with port 5000(we can use any unused port) which we need to specify in the args as arg['C'] is the port server is listening to.

***Stage 5 clear!***

locally all stages have been cleared, but on the [pwable.kr](http://pwable.kr) server we will need to tweak some things to make it work.

first of all to create our c script we will need to switch to /tmp and also for some reason (TODO: find why) we also need to create a dir inside it to make everything work.

There is another problem, /bin/cat flag will not work cz flag is not in our directory so we will need to create a symbolic link to flag using `ln -sf /home/input2/flag flag`

Also change `./input` to `/home/input2/input`

```c
input2@pwnable:/**tmp/walt**$ vi sc.c
input2@pwnable:/tmp/walt$ **ln -sf /home/input2/flag flag**
input2@pwnable:/tmp/walt$ cat flag
cat: flag: Permission denied
input2@pwnable:/tmp/walt$ gcc sc.c
input2@pwnable:/tmp/walt$ ./a.out
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
Mommy! ***************flag*******
input2@pwnable:/tmp/walt$
```

## Final script code

```c
#include <stdio.h>
#include<unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main()
{
	//stage 1
	char *arg[101] = {[0 ... 99] = "a"};
	arg['A'] = "\x00";
	arg['B'] = "\x20\x0a\x0d";
	arg['C'] = "5000";
	arg[100] = NULL;

	//stage 3
	char *envar[2] = {"\xde\xad\xbe\xef=\xca\xfe\xba\xbe",NULL};

	//stage 4
	FILE *fp = fopen("\x0a","w");
	fwrite("\x00\x00\x00\x00",4,1,fp);
	fclose(fp);

	//stage 2
	int pipetostdin[2],pipetostderr[2];

	if(pipe(pipetostdin)<0 || pipe(pipetostderr)<0)
		{
			printf("failed to create pipe\n");
		}

	pid_t childpid = fork();

	if(childpid < 0 )
	{
		printf("failed to fork\n");
	}

	
	if(childpid == 0)
	{
		//inside child process

		close(pipetostderr[0]); //close read pipes
		close(pipetostdin[0]);

		write(pipetostdin[1],"\x00\x0a\x00\xff", 4);
		write(pipetostderr[1],"\x00\x0a\x02\xff", 4);

		close(pipetostdin[1]);	// close write pipes
		close(pipetostderr[1]);

		//stage 5
		sleep(5);

		int sd;
		struct sockaddr_in saddr;
		sd = socket(AF_INET,SOCK_STREAM,0);
		if(sd == -1){
		printf("socket error\n");
		return 0;
		}
		saddr.sin_family = AF_INET;
		saddr.sin_addr.s_addr = inet_addr("127.0.0.1");
		saddr.sin_port = htons( 5000);
		if(connect(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("connect error, use another port\n");
    		return 1;
		}

		write(sd,"\xde\xad\xbe\xef", 4);

		close(sd);

	}
	else
	{
		//inside parent process

		close(pipetostdin[1]);	// close write pipes
		close(pipetostderr[1]);

		dup2(pipetostdin[0],0); //map read pipes to stdin and stderr
		dup2(pipetostderr[0],2);

		close(pipetostderr[0]); //close read pipes
		close(pipetostdin[0]);

		execve("/home/input2/input",arg,envar);
	}
	return 0;
}
```

![Untitled](/assets/images/posts/input//Untitled.png)

**References:**

- [https://jaimelightfoot.com/blog/pwnable-kr-input-walkthrough/](https://jaimelightfoot.com/blog/pwnable-kr-input-walkthrough/)

- [https://werewblog.wordpress.com/2016/01/11/pwnable-kr-input/](https://werewblog.wordpress.com/2016/01/11/pwnable-kr-input/)