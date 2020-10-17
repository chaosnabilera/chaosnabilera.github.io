---
title: "codeforces.com - 7D"
date: 2020-10-17
categories:
  - Problem Solving
tags:
  - codeforces.com
  - problem solving
---

[codeforces 2B]: http://codeforces.com/problemset/problem/7/D
[manachers algorithm]: https://en.wikipedia.org/wiki/Longest_palindromic_substring
[rabin karp algorithm]: https://en.wikipedia.org/wiki/Rabin%E2%80%93Karp_algorithm
[rolling hash]: https://en.wikipedia.org/wiki/Rolling_hash
[modular arithmetic]: https://en.wikipedia.org/wiki/Modular_arithmetic

[codeforces.com 2B Palindrome Degree][codeforces 2B] 문제풀이

# 문제
- k-palindrome 이란 그 자신이 palindrome이고 그 절반이 k-1 palindrome인 string을 말한다
    - 빈 string은 0-palindrome
    - 한글자는 1-palindrome
    - aa, aba는 2-palindrome
    - abcba는 1-palindrome
- 주어진 string의 prefix에서 얻을 수 있는 모든 k-palindrome의 숫자의 총합을 구하라

# 풀이

## Palindrome을 구하는 법
- Palindrome을 구하는 naive solution은 처음과 끝에서 한글자씩 매칭해 보는 것이다
- 따라서 모든 prefix의 palindrome 여부를 구하려면 N^2 의 시간이 걸린다
	- prefix라서 N^2인거고 substring이었으면 N^3이 걸렸을 것이다...
- 그런데 이 문제는 N이 최대 500만이므로 N^2 으로는 1초 안에 풀수 없다.
- 구하려고 하는게 palindrome이라는 점을 이용해서 좀 더 빠르게 - N이나 NlogN - 풀수 있는 방법이 있는지에 대해 열심히 생각해 보았으나 뾰족한 수는 나오지 않았다
- 결국 인터넷에서 palindrome algorithm에 대해 검색해 보다 [Manacher's algorithm][manachers algorithm] 과 [Rabin-Karp algorithm][rabin karp algorithm], [Rolling hash][rolling hash] 에 대해 알게 되어 실마리를 찾음. 실제로는 Rabin-Karp + Rolling hash로 해결하였다
	- Manacher's algorithm은 딱 보고 바로 이해가 안되서 그랬다. 언젠가는 이것도 한번 공부해 봐야 할 것 같다...

## Polynomial Rolling Hash
- ![](/assets/images/codeforces_7D_rollinghash.jpg)
- 위의 식에 mod m 연산을 하는 것이 Polynomial Rolling hash
	- a는 alphabet 보다 큰 prime number
	- m 은 큰 prime number (e.g. 1e9+9)
	- string을 a진수의 숫자로 표현한 뒤 그것을 mod m 했다고 생각하면 된다
	- k-1 제곱 에서 시작해도 되고 0 제곱부터 시작해서 하나씩 올려도 됨 (나는 이렇게 함)
- 실험해 보면 k와 m을 잘 선택하면 생각 이상으로 collision이 잘 안일어난다 (character를 하나씩 랜덤하게 붙일때 20만 정도에 한번씩 일어나는 느낌)
- 이 hash는 다음과 같은 장점이 있다
	- string에 대한 hash를 incremental 하게 구할 수 있음. 한 character를 더할때 마다 그냥 char X a^k+1 만 더해주면 되니까
	- 이것을 partial sum처럼 생각하면 substring에 대한 hash를 linear 하게 구할 수 있다. 예를 들어 abcde의 hash를 이미 구했다면 cd의 hash는 hash(1,4) - hash(1,2) 인 것이다
	- 주의해야 할 점은 다른 hash랑 비교할때 k를 곱해줄 필요가 있다는 것.
		- 예: aabbaabb
			- 첫 bb의 hash   = (b * a^2 + b * a^3) % m
			- 나중 bb의 hash = (b * a^6 + b * a^7) % m
			- 이런 경우 양측이 같은지 비교하려면 첫 bb의 hash에 a^(6-2) 만큼 곱해줘야 함
			- modular arithmetic은 곱셈에 대해 compatible 하므로 a^4 % m을 첫 bb의 hash에 곱해주면 됨
				- [Modular Arithmetic 참조][modular arithmetic]
- 이 hash를 이용하면 string 내부의 길이가 같은 substring이 일치하는 것을 O(1)에 구할 수 있음

## Rolling hash를 이용해서 prefix의 palindrome여부 구하기
- string을 왼쪽에서 오른쪽으로 읽으며 rolling hash를 구한다. 이를 HL이라 한다
- string을 오른쪽에서 왼쪽으로 읽으며 rolling hash를 구한다. 이를 HR이라 한다
- index는 0이 아니라 1부터 센다고 가정
- HL(1,i) == HR(i+1,i+i) 이거나 HL(0,i) == HR(i+1+1,i+1+i) 가 성립하는 것들을 찾는다 (위에서 설명한대로 알맞은 a를 곱하는 것을 잊지 말것)
- 이렇게 하면 O(N)에 모든 palindrome prefix를 구할 수 있다 (hash collision이 일어나지 않는다면!)

## k palindrome구하기
- 이건 그냥 아무 1-palindrome을 구했을때 이 것을 재로로 하는 2,3,...k-palindrome이 있는지 체크해 보면 된다

## 처음에 같은 숫자가 연속되는 것에 대한 처리
- 그냥 rolling hash를 이용해도 솔직히 상관은 없겠지만 이 부분은 그냥 간단하게 처리할 수 있다.
- 첫 글자는 1 palindrome, 그 다음 2개는 2-palindrome, 그 다음 4개는 3-palindrome 등등등 인 식

# 구현
- 메모리를 많이 잡아먹을 수도 있을 것 같아 C++로 구현함
- hash 관련 operation에서는 long long을 사용함