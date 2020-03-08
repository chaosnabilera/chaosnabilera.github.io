---
title: "pwnable.kr - Toddler's bottle #7 leg"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #7 leg 문제 풀이

leg.asm
```
(gdb) disass main
Dump of assembler code for function main:
   0x00008d3c <+0>:	push	{r4, r11, lr}
   0x00008d40 <+4>:	add	r11, sp, #8
   0x00008d44 <+8>:	sub	sp, sp, #12
   0x00008d48 <+12>:	mov	r3, #0
   0x00008d4c <+16>:	str	r3, [r11, #-16]
   0x00008d50 <+20>:	ldr	r0, [pc, #104]	; 0x8dc0 <main+132>
   0x00008d54 <+24>:	bl	0xfb6c <printf>
   0x00008d58 <+28>:	sub	r3, r11, #16
   0x00008d5c <+32>:	ldr	r0, [pc, #96]	; 0x8dc4 <main+136>
   0x00008d60 <+36>:	mov	r1, r3
   0x00008d64 <+40>:	bl	0xfbd8 <__isoc99_scanf>
   0x00008d68 <+44>:	bl	0x8cd4 <key1>
   0x00008d6c <+48>:	mov	r4, r0
   0x00008d70 <+52>:	bl	0x8cf0 <key2>
   0x00008d74 <+56>:	mov	r3, r0
   0x00008d78 <+60>:	add	r4, r4, r3
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0
   0x00008d84 <+72>:	add	r2, r4, r3
   0x00008d88 <+76>:	ldr	r3, [r11, #-16]
   0x00008d8c <+80>:	cmp	r2, r3
   0x00008d90 <+84>:	bne	0x8da8 <main+108>
   0x00008d94 <+88>:	ldr	r0, [pc, #44]	; 0x8dc8 <main+140>
   0x00008d98 <+92>:	bl	0x1050c <puts>
   0x00008d9c <+96>:	ldr	r0, [pc, #40]	; 0x8dcc <main+144>
   0x00008da0 <+100>:	bl	0xf89c <system>
   0x00008da4 <+104>:	b	0x8db0 <main+116>
   0x00008da8 <+108>:	ldr	r0, [pc, #32]	; 0x8dd0 <main+148>
   0x00008dac <+112>:	bl	0x1050c <puts>
   0x00008db0 <+116>:	mov	r3, #0
   0x00008db4 <+120>:	mov	r0, r3
   0x00008db8 <+124>:	sub	sp, r11, #8
   0x00008dbc <+128>:	pop	{r4, r11, pc}
   0x00008dc0 <+132>:	andeq	r10, r6, r12, lsl #9
   0x00008dc4 <+136>:	andeq	r10, r6, r12, lsr #9
   0x00008dc8 <+140>:			; <UNDEFINED> instruction: 0x0006a4b0
   0x00008dcc <+144>:			; <UNDEFINED> instruction: 0x0006a4bc
   0x00008dd0 <+148>:	andeq	r10, r6, r4, asr #9
End of assembler dump.
(gdb) disass key1
Dump of assembler code for function key1:
   0x00008cd4 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cd8 <+4>:	add	r11, sp, #0
   0x00008cdc <+8>:	mov	r3, pc
   0x00008ce0 <+12>:	mov	r0, r3
   0x00008ce4 <+16>:	sub	sp, r11, #0
   0x00008ce8 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008cec <+24>:	bx	lr
End of assembler dump.
(gdb) disass key2
Dump of assembler code for function key2:
   0x00008cf0 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cf4 <+4>:	add	r11, sp, #0
   0x00008cf8 <+8>:	push	{r6}		; (str r6, [sp, #-4]!)
   0x00008cfc <+12>:	add	r6, pc, #1
   0x00008d00 <+16>:	bx	r6
   0x00008d04 <+20>:	mov	r3, pc
   0x00008d06 <+22>:	adds	r3, #4
   0x00008d08 <+24>:	push	{r3}
   0x00008d0a <+26>:	pop	{pc}
   0x00008d0c <+28>:	pop	{r6}		; (ldr r6, [sp], #4)
   0x00008d10 <+32>:	mov	r0, r3
   0x00008d14 <+36>:	sub	sp, r11, #0
   0x00008d18 <+40>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d1c <+44>:	bx	lr
End of assembler dump.
(gdb) disass key3
Dump of assembler code for function key3:
   0x00008d20 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008d24 <+4>:	add	r11, sp, #0
   0x00008d28 <+8>:	mov	r3, lr
   0x00008d2c <+12>:	mov	r0, r3
   0x00008d30 <+16>:	sub	sp, r11, #0
   0x00008d34 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d38 <+24>:	bx	lr
End of assembler dump.
(gdb) 

```

ARM asm을 보고 이해해야 하는 문제. 일단 배경지식들은 이렇다.

1. ARM stated / Thumb state 에 따른 program counter 위치 차이
	- ARM state
		- the value of the PC is the address of the current instruction plus 8 bytes.
	- Thumb state
		- For B, BL, CBNZ, and CBZ instructions, the value of the PC is the address of the current instruction plus 4 bytes.
		- For all other instructions that use labels, the value of the PC is the address of the current instruction plus 4 bytes, with bit[1] of the result cleared to 0 to make it word-aligned.


2. ARM 모드인지 Thumb 모드인지 구분하는 법
	- The easiest way to check if an address is considered to be ARM or Thumb is to check the disassembly. If the instruction is 32 bits wide – it is ARM, if it is 16 bits wide it is Thumb.

3. BX/BLX와 ARM/Thumb
	- BX Rm and BLX Rm derive the target state from bit[0] of Rm:
		- if bit[0] of Rm is 0, the processor changes to, or remains in, ARM state
		- if bit[0] of Rm is 1, the processor changes to, or remains in, Thumb state.

4. asm을 보면 r0가 함수 리턴 값이다

### key1
* mov r3, pc
	* pc (program counter)를 r3에 넣는다
~~~
   0x00008cdc <+8>:	    mov	r3, pc
   0x00008ce0 <+12>:	mov	r0, r3
~~~

* instruction이 4byte wide이니 ARM state인듯?
* 이걸 보건대 리턴 값(=r0)은 0x8cdc + 0x8 = 0x8CE4이다

### key2
* push {r6}
	* r6 값을 stack에 push
* add r6, pc, $1
	* r6에 pc+1을 넣는다 (r6+pc+1이 아니다!)
	* 이때 pc = 0x00008cfc + 0x8 = 0x8d04
	* 그럼 r6에는 0x8d05가 들어간다
	* 1 더하는 이유가 이해가 안되었는데 보니까 BX일때 1이 있으면 Thumb state라는 뜻이 되는 것 같다
* bx r6
	* bx : branch indirect + ARM <-> Thumb (이 경우는 ARM모드였으니 Thumb으로 간다)
		* r6로 가라 (1을 제외하고 보는듯? 즉 0x8d04로 가라)
		* 0x8d04는 결국 mov r3,pc니까 결국 다음으로 넘어가는거다...
	* bl 은 분기하되 return address를 r14에 저장한다 라는 뜻
* .code 16
    * 여기서 부터는 16bit (thumb) 모드로 짜져 있다
* mov r3, pc
    * r3에 pc를 넣어라
    * thumb mode니까 pc = 0x00008d04 + 4 = 0x8d08이다
* add	r3, $0x4
    * r3에 4를 더해라
    * r3 = 0x8d08 + 4 = 0x8d0c
* push	{r3}
	* r3를 push 해라
* pop	{pc}
	* pc에다 pop 해라
	* 이러면 r3 = 0x8d0c로 가는건데 그게 그냥 아래의 pop {r6} 하는 부분임
* .code	32
	* 여기서 부터는 32bit (ARM) 모드다
* pop	{r6}
	* r6에 pop해라
* epilogue 보면 마지막에 r3 = 0x8d0c를 r0에 넣는다 즉 리턴값은 0x8d0c

### key3
* mov r3, lr
	* lr을 r3에 넣어라
	* lr은 r14, 즉 리턴 어드레스가 있는 곳이다. key3의 경우는 0x8d80

### 답
0x8ce4 + 0x8d0c + 0x8d80 = 0x1A770‬ = 108400

~~~
/ $ ./leg
Daddy has very strong arm! : 108400
Congratz!
<tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- My daddy has a lot of ARMv5te muscle! -->