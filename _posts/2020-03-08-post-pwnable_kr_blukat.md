---
title: "pwnable.kr - Toddler's bottle #19 blukat"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #19 blukat 문제 풀이

~~~
Sometimes, pwnable is strange...
hint: if this challenge is hard, you are a skilled player.
~~~

일단 문제를 딱 보면 당황 스럽다. 이걸갖고 뭘 어떻게 하지?

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
char flag[100];
char password[100];
char* key = "3\rG[S/%\x1c\x1d#0?\rIS\x0f\x1c\x1d\x18;,4\x1b\x00\x1bp;5\x0b\x1b\x08\x45+";
void calc_flag(char* s){
        int i;
        for(i=0; i<strlen(s); i++){
                flag[i] = s[i] ^ key[i];
        }
        printf("%s\n", flag);
}
int main(){
        FILE* fp = fopen("/home/blukat/password", "r");
        fgets(password, 100, fp);
        char buf[100];
        printf("guess the password!\n");
        fgets(buf, 128, stdin);
        if(!strcmp(password, buf)){
                printf("congrats! here is your flag: ");
                calc_flag(password);
        }
        else{
                printf("wrong guess!\n");
                exit(0);
        }
        return 0;
}
```

일단 bof는 아니다. Segmentation fault가 안나니
~~~
blukat@pwnable:~$ python -c "print 'a'*128" | ./blukat
guess the password!
wrong guess!
blukat@pwnable:~$ python -c "print 'a'*208" | ./blukat
guess the password!
wrong guess!
~~~

저 key를 그대로 집어넣으면 되나? 싶었는데 역시 안된다
~~~
blukat@pwnable:~$ printf "\x33\x0D\x47\x5B\x53\x2F\x25\x1C\x1D\x23\x30\x3F\x0D\x49\x53\x0F\x1C\x1D\x18\x3B\x2C\x34\x1B\x00\x1B\x70\x3B\x35\x0B\x1B\x08\x45\x2B" | ./blukat
guess the password!
wrong guess!
~~~

뭐지? 그냥 cat했을때 이렇게 나온다
~~~
blukat@pwnable:~$ cat /home/blukat/password
cat: password: Permission denied
~~~

힌트에 보면 이게 어려우면 skilled라고 하는데... 난 skilled가 아닌데... 그럼 뭔가 아주 쉬운 방법이 있다는 건가? 라고 생각하고 있었는데... 가만 보니 저 cat 메시지 자체가 32자다. 그리고 password내용은 33자. string이라면 ending null이 포함되어 33자일거라 내용 자체는 32자라는 뜻일것이다.

그렇다면 혹시 속임수 문제인가? 하고 저 cat메시지를 그대로 넣어보았다

~~~
blukat@pwnable:~$ ./blukat
guess the password!
cat: password: Permission denied
congrats! here is your flag: <tag output>
~~~

flag를 얻었다. 근데 왠 group perm? 하고 id를 쳐보니....

~~~
blukat@pwnable:~$ ls -als
total 36
 4 drwxr-x---   4 root blukat     4096 Aug 16  2018 .
 4 drwxr-xr-x 116 root root       4096 Nov 12 21:34 ..
12 -r-xr-sr-x   1 root blukat_pwn 9144 Aug  8  2018 blukat
 4 -rw-r--r--   1 root root        645 Aug  8  2018 blukat.c
 4 dr-xr-xr-x   2 root root       4096 Aug 16  2018 .irssi
 4 -rw-r-----   1 root blukat_pwn   33 Jan  6  2017 password
 4 drwxr-xr-x   2 root root       4096 Aug 16  2018 .pwntools-cache
blukat@pwnable:~$ id
uid=1104(blukat) gid=1104(blukat) groups=1104(blukat),1105(blukat_pwn)
~~~

약간 넌센스 문제였다...

[pwnable.kr]: https://pwnable.kr

<!-- Pl3as_DonT_Miss_youR_GrouP_Perm!! -->