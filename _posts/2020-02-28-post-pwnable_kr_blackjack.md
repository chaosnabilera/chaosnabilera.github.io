---
title: "pwnable.kr - Toddler's bottle #11 blackjack"
date: 2020-02-28
categories:
  - blog
tags:
  - pwnable.kr
---

[pwnable.kr][pwnable.kr] Toddler's bottle의 #10 blackjack 문제 풀이

처음엔 그냥 이겨야 되는건가 싶었는데 $8000까지 벌었다가 졌다...

pwnable이니까 뭔가 다른 방법이 있겠지 싶어 이거저거 해보다 결국엔 Integer overflow 문제라는걸 발견했다.

~~~
Enter Bet: $100000000

You cannot bet more money than you have.
Enter Bet: 9999999999


Would You Like to Hit or Stay?
Please Enter H to Hit or S to Stay.
H
... 

YaY_I_AM_A_MILLIONARE_LOL

Cash: $1410065907
~~~

[pwnable.kr]: https://pwnable.kr