---
title: "pwnable.kr - Toddler's bottle #3 flag"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #3 flag 문제 풀이

all you need is binary

가져와서 ghidra를 돌려보면 ELF / x86:LE:64:default:gcc 라고 나온다

gdb로 볼라고 하면 심볼이 없어서 이런 메시지를 뱉음
~~~
(gdb) b main
No symbol table is loaded.  Use the "file" command.
~~~

dll이 malloc하고 strcpy 한다길래 so 링크할때 스탑하면 되나? 했는데 안됨
~~~
(gdb) show stop-on-solib-events
Stopping for shared library events is 0.
(gdb) set stop-on-solib-events 1
(gdb) run
Starting program: /home/vmware/flag 
I will malloc() and strcpy the flag there. take it.
[Inferior 1 (process 10608) exited normally]
~~~

info file 해봐도
~~~
(gdb) info file
Symbols from "/home/vmware/flag".
~~~

obj dump 해보면 entry는 나온다
~~~
vmware@ubuntu:~$ objdump -f flag

flag:     file format elf64-x86-64
architecture: i386:x86-64, flags 0x00000102:
EXEC_P, D_PAGED
start address 0x000000000044a4f0
~~~

~~~
vmware@ubuntu:~$ gdb flag

(gdb) b *0x44a4f0
Breakpoint 1 at 0x44a4f0
~~~

저렇게 entry에서 그냥 disas 하면 이렇게 나온다
~~~
(gdb) disas
No function contains program counter for selected frame.
(gdb) disas *0x44a4f0
No function contains specified address.
~~~

이게 일어나는 이유가 disas가 원래 default는 code of current function을 disas해주는데 지금 function의 시작과 끝을 확실히 알수 없어서 이렇게 나오는 거다. 그래서 아래와 같이 어디부터 어디까지 disas 할건지 manual하게 알려줘야 한다

~~~
(gdb) disas $rip, +16
Dump of assembler code from 0x44a4f0 to 0x44a500:
=> 0x000000000044a4f0:	callq  0x44a770
   0x000000000044a4f5:	push   %rbp
   0x000000000044a4f6:	push   %rbx
   0x000000000044a4f7:	push   %rcx
   0x000000000044a4f8:	push   %rdx
   0x000000000044a4f9:	add    %rdi,%rsi
   0x000000000044a4fc:	push   %rsi
   0x000000000044a4fd:	mov    %rdi,%rsi

~~~

근데 여기까지 보고 나면 뭔가 이상하다는 생각이 든다. 따라가 보면 막 코드를 만들어 내고 점프하고 calling convention도 안지키고 이상하다
여기까지 보고 pack 되어 있을지도 모른다는 생각을 할 수 있어야 한다는게 포인트인듯...
Detect It Easy라는 툴이 있다. 만약 Pack 되어 있다면 이것으로 체크해 볼 수 있다
돌려보면 UPX(3.08)[NRV,brute]라고 뜸
UPX unpack 후 gdb를 돌려보면 잘 나온다

~~~
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:	push   %rbp
   0x0000000000401165 <+1>:	mov    %rsp,%rbp
   0x0000000000401168 <+4>:	sub    $0x10,%rsp
   0x000000000040116c <+8>:	mov    $0x496658,%edi
   0x0000000000401171 <+13>:	callq  0x402080 <puts>
   0x0000000000401176 <+18>:	mov    $0x64,%edi
   0x000000000040117b <+23>:	callq  0x4099d0 <malloc>
   0x0000000000401180 <+28>:	mov    %rax,-0x8(%rbp)
   0x0000000000401184 <+32>:	mov    0x2c0ee5(%rip),%rdx        # 0x6c2070 <flag>
   0x000000000040118b <+39>:	mov    -0x8(%rbp),%rax
   0x000000000040118f <+43>:	mov    %rdx,%rsi
   0x0000000000401192 <+46>:	mov    %rax,%rdi
   0x0000000000401195 <+49>:	callq  0x400320
   0x000000000040119a <+54>:	mov    $0x0,%eax
   0x000000000040119f <+59>:	leaveq 
   0x00000000004011a0 <+60>:	retq   
End of assembler dump.
(gdb) b strcpy
Breakpoint 1 at gnu-indirect-function resolver at 0x40c050
(gdb) r
Starting program: /home/vmware/flag 
I will malloc() and strcpy the flag there. take it.

Breakpoint 1, 0x0000000000416b50 in __strcpy_sse2_unaligned ()
(gdb) x/s $rdx
0x496628:	<tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- UPX...? sounds like a delivery service :) -->