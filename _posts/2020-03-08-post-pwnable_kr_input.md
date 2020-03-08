---
title: "pwnable.kr - Toddler's bottle #6 input"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #6 input 문제 풀이

input.c
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

짜서 풀어야 하는 문제다. 나는 python을 사용하였다

stage1_4.py
```python
import os
import subprocess
import sys

iArgv = ['a' for i in range(100)]
iArgv[0] = "/home/input2/input"

# argv
iArgv[ord('A')] = ""   # strcmp "\x00" = strcmp ""
iArgv[ord('B')] = "\x20\x0a\x0d"

# stdio
stdinF = open("stdinF","wb")
stdinF.write("\x00\x0a\x00\xff")
stdinF.close()

stdoutF = open("stdoutF","wb")
stdoutF.write("\x00\x0a\x02\xff")
stdoutF.close()

# env
os.environ["\xde\xad\xbe\xef"] = "\xca\xfe\xba\xbe"

# file
fileF = open("\x0a","wb")
fileF.write("\x00\x00\x00\x00")
fileF.close()

# network
iArgv[ord('C')] = sys.argv[1]

subprocess.Popen(iArgv, stdin=open("stdinF","rb"), stderr=open("stdoutF","rb"))
```

stage5.py
```python
import socket
import sys

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('127.0.0.1',int(sys.argv[1])))
sock.sendall("\xde\xad\xbe\xef")
sock.close()
```

참고로 마지막에 /bin/cat flag 하는데 python은 write할수 있는 directory에서 돌아야 해서 flag에 대한 symbolic link를 만들어야 한다

~~~
ln -s /home/input2/flag flag
...
input2@pwnable:/tmp/cn$ python stage1_4.py 60029
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
input2@pwnable:/tmp/cn$ Stage 5 clear!
<tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- Mommy! I learned how to pass various input in Linux :) -->