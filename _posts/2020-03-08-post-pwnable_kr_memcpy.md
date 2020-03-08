---
title: "pwnable.kr - Toddler's bottle #16 memcpy"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #16 memcpy 문제 풀이

풀긴 풀었는데 막 집어넣다 보니 얼떨결에 풀어서 다른 풀이를 보고서야 왜 이렇게 된건지 이해했다. 사실 이것도 대략만 이해했다...

소스 코드를 보면 fast_memcpy는 아래와 같은 명령어를 사용한다

```c
char* fast_memcpy(char* dest, const char* src, size_t len){
	size_t i;
	// 64-byte block fast copy
	if(len >= 64){
		i = len / 64;
		len &= (64-1);
		while(i-- > 0){
			__asm__ __volatile__ (
			"movdqa (%0), %%xmm0\n"
			"movdqa 16(%0), %%xmm1\n"
			"movdqa 32(%0), %%xmm2\n"
			"movdqa 48(%0), %%xmm3\n"
			"movntps %%xmm0, (%1)\n"
			"movntps %%xmm1, 16(%1)\n"
			"movntps %%xmm2, 32(%1)\n"
			"movntps %%xmm3, 48(%1)\n"
			::"r"(src),"r"(dest):"memory");
			dest += 64;
			src += 64;
		}
	}

	// byte-to-byte slow copy
	if(len) slow_memcpy(dest, src, len);
	return dest;
}
```

여기에서 movdqa는 이런 놈이다
~~~
Moves a double quadword from the source operand (second operand) to the destination operand (first operand). 
...
When the source or destination operand is a memory operand, the operand must be aligned on a 16-byte boundary or a general-protection exception (#GP) will be generated.
~~~

그냥 실행해 보면 아래와 같은 결과가 나온다
~~~
ellapsed CPU cycles for slow_memcpy : 5366
ellapsed CPU cycles for fast_memcpy : 494

experiment 2 : memcpy with buffer size 16
ellapsed CPU cycles for slow_memcpy : 634
ellapsed CPU cycles for fast_memcpy : 596

experiment 3 : memcpy with buffer size 32
ellapsed CPU cycles for slow_memcpy : 604
ellapsed CPU cycles for fast_memcpy : 864

experiment 4 : memcpy with buffer size 64
ellapsed CPU cycles for slow_memcpy : 1250
ellapsed CPU cycles for fast_memcpy : 420

experiment 5 : memcpy with buffer size 128
ellapsed CPU cycles for slow_memcpy : 2008
memcpy@pwnable:~$ 
~~~

즉 중간에 뻗는다

나는 이게 뭐지? 하고 128에서 숫자를 더하고 빼보다가 8을 더하니 갑자기 256까지 가는걸 봤다.
??? 하면서도 일단 그 다음것도 계속 8을 더하다 보니 결국 문제를 해결하게 됐다

왜 이런문제가 발생했는가 하니, 소스에 보면 dst 가 malloc에 의해 할당이 되는데, malloc은 할당을 할때 앞에 얼마나 할당 됐는지를 알려주는 meta-chunk를 삽입한다

https://sourceware.org/glibc/wiki/MallocInternals

여기에 이렇게 설명되어 있다

~~~
...
Each chunk includes meta-data about how big it is (via a size field in the chunk header), and thus where the adjacent chunks are. When a chunk is in use by the application, the only data that's "remembered" is the size of the chunk. 
...
Since all chunks are multiples of 8 bytes, the 3 LSBs of the chunk size can be used for flags. These three flags are defined as follows:
~~~

다시 말해 사용중인 malloc chunk의 맨 앞에는 size가 붙고, 이 size는 언제나 8byte다.

그런데 malloc의 alignment는 32bit는 8, i386은 16, 64bit는 16이다.

소스에 보면 
~~~
// compiled with : gcc -o memcpy memcpy.c -m32 -lm
~~~

이렇게 나와 있는데 여기에서 -m32는 32bit 모드로 컴파일 한다는 뜻이다. 그렇다면 alignment가 8 byte 단위로 이루어질 것이므로

8 ->  16 ->   32 -> 64->128 하면 실제로는

16 ->  24 ->   40 -> 72 -> 130 이렇게 allocate되었을 것이고 그렇다면 alignment가

8 -> 16 -> 8 -> 16 -> 8 이랬을 것이다.

그래서 64일때는 16byte alignment라 멀쩡히 통과한거고 128일때는 8byte라 통과하지 못한 것이다.

결국 메모리 alignment가 중요하다는 이야기

~~~
experiment 1 : memcpy with buffer size 8
ellapsed CPU cycles for slow_memcpy : 5248
ellapsed CPU cycles for fast_memcpy : 496

experiment 2 : memcpy with buffer size 16
ellapsed CPU cycles for slow_memcpy : 546
ellapsed CPU cycles for fast_memcpy : 886

experiment 3 : memcpy with buffer size 32
ellapsed CPU cycles for slow_memcpy : 706
ellapsed CPU cycles for fast_memcpy : 832

experiment 4 : memcpy with buffer size 72
ellapsed CPU cycles for slow_memcpy : 1206
ellapsed CPU cycles for fast_memcpy : 522

experiment 5 : memcpy with buffer size 136
ellapsed CPU cycles for slow_memcpy : 3646
ellapsed CPU cycles for fast_memcpy : 436

experiment 6 : memcpy with buffer size 264
ellapsed CPU cycles for slow_memcpy : 4076
ellapsed CPU cycles for fast_memcpy : 764

experiment 7 : memcpy with buffer size 520
ellapsed CPU cycles for slow_memcpy : 7510
ellapsed CPU cycles for fast_memcpy : 614

experiment 8 : memcpy with buffer size 1032
ellapsed CPU cycles for slow_memcpy : 15010
ellapsed CPU cycles for fast_memcpy : 794

experiment 9 : memcpy with buffer size 2056
ellapsed CPU cycles for slow_memcpy : 29198
ellapsed CPU cycles for fast_memcpy : 1512

experiment 10 : memcpy with buffer size 4096
ellapsed CPU cycles for slow_memcpy : 57722
ellapsed CPU cycles for fast_memcpy : 2376

thanks for helping my experiment!
flag : <tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- 1_w4nn4_br34K_th3_m3m0ry_4lignm3nt -->