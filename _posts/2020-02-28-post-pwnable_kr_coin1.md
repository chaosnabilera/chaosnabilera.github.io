---
title: "pwnable.kr - Toddler's bottle #10 coin1"
date: 2020-02-26
categories:
  - blog
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #10 coin1 문제 풀이

N개의 coin이 주어지고 C번의 trial 안에 몇번째 coin이 fake coin인지 판별하는 문제다.

진짜 coin은 무게가 10이고 가짜 coin은 9다.

한번의 trial마다 x개의 coin들의 무게 총합에 대한 query를 날릴수 있다.

결국 binary search 문제다.

60초 안에 100개를 찾아야 하므로 손으로는 못한다. 나는 python으로 짜서 풀었다.

외부에서 접속하면 latency가 느려서 60초 안에 100번 깨는게 안된다.

그래서 아무 pwnable.kr 계정으로나 들어가서 서버 안에서 파이썬을 돌리는걸 통해 해결했다

```python
import re
import socket
import time

addr = ('pwnable.kr',9007)

cs = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
cs.connect(addr)

data = cs.recv(4096)
print(data.decode('utf-8'))

time.sleep(4)

while True:
	problem = cs.recv(4096).decode('utf-8').strip()
	print(problem)
	
	N,C = re.findall("N=(\d+) C=(\d+)", problem)[0]
	N,C = int(N), int(C)
	
	s = 0
	e = N

	while True:
		print('s',s,'e',e)
		cStart = s
		#if (e-s) % 2 == 0:
		if e > s+1:
			cEnd = cStart + ((e-s)//2)
		else:
			cEnd = e

		#print('cStart',cStart,'cEnd',cEnd)

		toSend = " ".join(str(c) for c in range(cStart, cEnd)) + "\n"
		toSend = toSend.encode("utf-8")

		#print('Q>',toSend)

		cs.sendall(toSend)

		result = cs.recv(4096).decode('utf-8').strip()

		print(result)
		if 'Correct' in result:
			break

		result = int(result)

		#print('A>',result)

		if result == 10 * (cEnd - cStart):
			s = cEnd
		else:
			e = cEnd
	
```

~~~
...
Correct! (99)
Congrats! get your flag
b1NaRy_S34rch1nG_1s_3asy_p3asy
~~~

[pwnable.kr]: https://pwnable.kr