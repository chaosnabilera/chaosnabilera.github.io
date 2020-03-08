---
title: "pwnable.kr - Toddler's bottle #5 random"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #5 random 문제 풀이

random.c
```c
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();        // random value!

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

rand() 에 대해 seed가 안되어 있어서 같은 값이 나온다

~~~
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();

        printf("%08X\n",random);
        return 0;
}
~~~

이렇게 컴파일 해보면 6B8B4567 가 나옴

~~~
>>> 0xDEADBEEF ^ 0x6B8B4567
3039230856
~~~

~~~
random@pwnable:~$ ./random
3039230856
Good!
<tag output>
~~~


[pwnable.kr]: https://pwnable.kr

<!-- Mommy, I thought libc random is unpredictable... -->