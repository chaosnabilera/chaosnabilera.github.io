---
title: "pwnable.kr - Toddler's bottle #1 collision"
date: 2020-03-08
categories:
  - Information Security
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #1 collision 문제 풀이

col에 setuid가 걸려있음
~~~
col@pwnable:~$ ls -als
total 36
4 drwxr-x---   5 root    col     4096 Oct 23  2016 .
4 drwxr-xr-x 116 root    root    4096 Nov 12 21:34 ..
4 d---------   2 root    root    4096 Jun 12  2014 .bash_history
8 -r-sr-x---   1 col_pwn col     7341 Jun 11  2014 col
4 -rw-r--r--   1 root    root     555 Jun 12  2014 col.c
4 -r--r-----   1 col_pwn col_pwn   52 Jun 11  2014 flag
4 dr-xr-xr-x   2 root    root    4096 Aug 20  2014 .irssi
4 drwxr-xr-x   2 root    root    4096 Oct 23  2016 .pwntools-cache
~~~

col.c를 보면 argv[1]에서 20byte를 읽은 다음 이걸 4byte int x 5로 해석하여 더한 값이 0x21DD09EC 와 일치하면 패스하게 되어 있다.

0x21DD09EC - (0x01010101)x4 = 0x1DD905E8

~~~
printf "\x01\x01\x01\x01\x01\x01\x01\x01\x01\x01\x01\x01\x01\x01\x01\x01\xE8\x05\xD9\x1D" > /tmp/soll
./col `cat /tmp/soll`
<tag output>
~~~

[pwnable.kr]: https://pwnable.kr

<!-- daddy! I just managed to create a hash collision :) -->