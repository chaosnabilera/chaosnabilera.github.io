---
title: "C++ 에서의 IO 에 대한 매우매우 간단한 조사"
date: 2021-08-01
categories:
  - C++
tags:
  - I/O
---

[https://www.acmicpc.net/blog/view/105]: https://www.acmicpc.net/blog/view/105
[https://panty.run/fastio/]: https://panty.run/fastio/

# TLE
- Fast IO의 세계는 생각보다 깊다. 쓰려면 쓰겠지만 좀 성가심...
- 그냥 cin/cout은 느리다. sync_with_stdio(false) 쓰면 빨라짐
- printf/scanf가 cin/cout + sync_with_stdio(false) 보다 살짝 느리긴 한데 비슷함. 여기에 굳이 신경써가면서 문제 풀고 싶지도 않기도 하고 이미 내가 익숙한 방식이니 당분간은 그냥 여기에 정착하는게 좋을것 같다. 이거나 쓰는게 속편할 듯?

# 목적
- codeforces 풀다 보니 Python 으로 푸는건 여러모로 한계가 있는 것 같다
  - Python 에는 sorted container가 잘 없다. 오픈소스 프로젝트로 만든것들이 있기는 한데 이게 leetcode에서는 되지만 codeforces에서는 안된다
  - 나는 분명 알고리즘을 제대로 짰는데 python으로 짜면 TLE가 걸리고 C++로 짜면 매우매우 여유 있게 통과하는 경우가 꽤 있다. 그것도 rating이 높은쪽으로 가면 갈수록 많아짐...
  - 난 솔직히 아주 높은 rating까지 가기는 여러모로 힘들것 같아서 적당히 python으로 뭉겔 생각이었는데 rating 1800짜리 문제가 저런식으로 걸리는 거 보고 역시 C++로 해야 하나... 라는 생각이 듦

- 근데 난 PS를 주로 python으로 했어서 C++로 짜는 것에 좀 저항감이 있음... 그래서 그 저항감도 줄일 겸 전환 과정에서 답답하게 느껴질 것 같은 것들을 좀 미리미리 해결하고 넘어갈 겸 해서 이렇게 정리를 한다

# Fast I/O
- 예전에 풀었던 문제들 중에서 cin/cout을 썼을때는 통과하지 못하고 printf/scanf를 썼을때는 통과했던 기억이 있어서 한번 짚고 넘어가기로 함
- 인터넷에 찾아 보면 꽤 이것저것 나옴
  - [https://www.acmicpc.net/blog/view/105][https://www.acmicpc.net/blog/view/105]
  - [https://panty.run/fastio/][https://panty.run/fastio/]
- 쭉 읽어보니 이건 나의 영역이 아닌것 같음... 뭐 정말 대용량 input 을 처리할때는 저기까지 가야겠지만 뭐 솔직히 그정도 수준까지 최적화가 필요한 문제는 풀 일이 없지 않을까...
- 있는 소스코드를 갖다 쓸수도 있겠지만... 뭔가 지는 느낌임 \~\_\~ ㄲㄲㄲ
- 그래서 그냥 cin/cout, cin/cout + sync_with_stdio(false), printf/scanf 나 비교해 보기로 함

# random number 생성
```python
import random
import sys

NUM_RND = 1000000

if __name__ == '__main__':
  if len(sys.argv) == 2:
    NUM_RND = int(sys.argv[1])

  print(NUM_RND)
  for _ in range(NUM_RND):
    print(random.randint(-1000000,1000000))
```
- 이 코드로 1000만개 생성해서 실험. 대략 71MB 였다

# cin/cout
```cpp
#include <iostream>

int main(){
  int N;
  int* nums;

  std::cin >> N;

  nums = new int[N];

  for(int i=0; i<N; ++i){
    std::cin >> nums[i];
  }

  std::cout << N;
  for(int i=0; i<N; ++i){
    std::cout << nums[i] << '\n';
  }
}
```
- g++로 컴파일
- average: 3.5616s

# cin/cout + sync_with_stdio(false)
```cpp
#include <iostream>

int main(){
  int N;
  int* nums;

  std::ios::sync_with_stdio(false);

  std::cin >> N;

  nums = new int[N];

  for(int i=0; i<N; ++i){
    std::cin >> nums[i];
  }

  std::cout << N;
  for(int i=0; i<N; ++i){
    std::cout << nums[i] << '\n';
  }
}
```
- average: 1.607s

# printf/scanf
```cpp
#include <iostream>

int main(){
  int N;
  int* nums;

  scanf("%d", &N);

  nums = new int[N];

  for(int i=0; i<N; ++i){
    scanf("%d", &nums[i]);
  }

  printf("%d\n", N);
  for(int i=0; i<N; ++i){
    printf("%d\n", nums[i]);
  }
}
```
- average: 1.7012s

# 결론?
- TLE에 적은 대로
- printf/scanf 에서 피할 수 없는 한계를 느끼기 전까지는 그냥 단순하고 익숙하고 쓰기 쉽고 적당히 빠른걸 쓰자. 예압.