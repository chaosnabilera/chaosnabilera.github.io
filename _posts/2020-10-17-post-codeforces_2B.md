---
title: "codeforces.com - 2B"
date: 2020-10-17
categories:
  - Problem Solving
tags:
  - codeforces.com
  - problem solving
---

[codeforces 2B]: http://codeforces.com/problemset/problem/2/B

[codeforces.com 2B The least round way][codeforces 2B] 문제풀이

# 문제
- N X N matrix가 주어진다
- 각 matrix의 숫자는 10^9 보다 작은 non-negative number다
- 왼쪽 위 끝에서 오른쪽 아래 끝으로 한번에 오른쪽으로 한칸 또는 아래로 한칸씩 움직일 수 있다
- 가능한 path중 그 path의 수를 모두 곱했을 때 끝에 붙는 0의 숫자를 최소화 하는 path를 구해라
- 0 최소 path에서 뒤에 붙는 0의 개수를 구해라

# 풀이

## 답의 형태
- 어떤 path든 오른쪽으로 N-1번 움직이고 아래로 N-1번 움직여야 한다. 순서의 차이가 있을 뿐이다
- path를 전부 곱한 숫자 R의 끝이 0으로 끝난다는 것은 다음을 의미한다
	- R = 0. 즉 path의 중간에 0이 있었음
	- R = a X 10^b / a>=1 / b >= 1
- 10 = 2 X 5 니까 path에 2와 5가 각각 몇번 나오는지가 중요하다

## 처음에 한 삽질 : 2와 5를 동시에 봐야 한다고 생각했음
- ![](/assets/images/codeforces_2B_wrong1.jpg)
- 이 formulation은 오른쪽 끝 column과 아래쪽 끝 row에 대해서는 옳다. 왜냐하면 오른쪽 끝에 도달하거나 아래쪽 끝으로 도달한 경우에는 선택지 없이 아래로만 내려가거나 오른쪽으로만 이동할 수 있기 때문.
- 하지만 이러한 생각은 틀렸다. 전 단계에서 0의 숫자를 최소화 했다고 해서 그것이 이번 단계에 유용하다는 보장이 없다는 점이다
	- 예를 들어 전 단계에서 한 선택지는 0의 숫자가 1개지만 남는 2가 100개 있고 다른 한 선택지는 0의 숫자가 10개지만 남는 2는 없다고 하자
	- 이번 단계에서 내가 5를 100개 들고 있다면 2가 100개 있는 path로 가면 안된다. 오히려 0은 더 많지만 2가 적은 path를 고르는 쪽이 0의 숫자를 최소화 할 수 있다

## 해답: 2가 최소화 되는 path, 5가 최소화 되는 path, 중간에 0을 곱하는 path 중 하나가 답이다
- ![](/assets/images/codeforces_2B_right.jpg)
- 이렇게 접근하면 임의의 prime number x의 숫자를 최소화 하는 path는 구할 수 있다
- 0의 숫자와의 차이점은 prime number는 그 자신의 숫자 외의 다른 요소에 영향을 받지 않는다는 것에 있다
- 직관적으로 생각해 봐도 0이 최소화 되려면 2가 가장 적던가 5가 가장 적던가 둘중의 하나여야 한다
- 좀 형식적인 증명은 다음과 같다
	- 2가 최소화 되었을때 2와 5의 개수가 a,b / 5가 최소화 되었을때 2와 5의 개수가 c,d라고 하자
	- 2가 최소화 되었을때 0의 개수는 min(a,b) 이고 5가 최소화 되었을때 끝의 0의 개수는 min(c,d) 이다
	- 2가 최소화 되었을때 보다 0이 더 적은 답이 있고 이때 2와 5의 개수가 각각 p,q 라고 하자. 
	- 이때 p > a, q < b, min(p,q) < min(a,b) 여야 한다. 
	- 그런데 5가 최소화 되었을 때 5의 개수인 d는 반드시 q 보다 작거나 같아야 한다. 즉 d < q < b 인 셈 
	- 이때 min(c,d) > min(p,q) 이려면 min(c,d) = c 여야한다.
	- 그런데 이때 min(c,d) >= min (a,b) 일 수 밖에 없다. a는 2를 최소화 했을때의 2의 개수이니 a는 c보다 작거나 같을 수 밖에 없다. 이렇게 되면 p,q는 모순일 수 밖에 없다.
	- p,q가 모순이 아니려면 min(c,d) <= min(p,q) 여야 한다.
	- 위와 같은 전개를 2와 5를 바꿔서도 가능하다
	- 즉 2가 최소화 된 path 또는 5가 최소화 된 path가 10이 최소화 된 path다
- 위에 더해서 중간에 0이 있다면 그 0을 지나가는게 끝의 0을 최소화 하는 것일 수 있다
- 즉 3중의 하나가 0 최소화 path 인것

# 구현
- 처음엔 python으로 했는데 DP 이다 보니 N 이 커지면 메모리 limit에 걸려서 C++ 로 해결함