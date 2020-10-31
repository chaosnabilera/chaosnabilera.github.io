---
title: "codeforces.com - 15D"
date: 2020-10-31
categories:
  - Problem Solving
tags:
  - codeforces.com
  - problem solving
---

[codeforces 15D]: http://codeforces.com/problemset/problem/15/D
[https://www.programmersought.com/article/8605496453/]: https://www.programmersought.com/article/8605496453/

[codeforces 15D Map][codeforces 15D] 문제풀이

# 문제
- nXm matrix에 non-negative number가 주어짐
- 1<=a<=n<=1000, 1<=b<=m<=1000
- 주어진 map의 임의의 공간 aXb 안에서 가장 작은 숫자를 min 이라고 하고 aXb안의 모든 숫자를 더한걸 sum이라고 하자. 그리고 cost = sum - (min X a X b) 라고 하자. 즉 cost는 a X b안에 있는걸 모두 min까지 줄인다고 했을때 총 얼마나 줄여야 하는지에 대한 숫자다.
- cost 가 가장 작은것 부터 하나씩 aXb만큼 채워나간다고 했을때, 그리고 겹치는 것이 없을 때 어떤 순서로 몇개를 채울수 있나?
- cost가 같은게 있으면 더 위에 있는게 우선되고 같은 높이에 있으면 더 왼쪽에 있는게 우선된다.

# 주의점
- 아래의 풀이1에는 꽤 빨리 도달했는데 데이터 입력을 std::cin으로 받는것 때문에 계속 실패했음...
	- 나름 sync_with_stdio(false)도 하고 했는데도 그랬다
	- 대략 비교해 보면 cin으로는 100만개의 데이터를 받는것 자체에 1.8초 정도 걸림. 반면 scanf로 받으면 0.5초도 안걸린다
	- PS 문제를 풀때는 fast io를 쓰는게 정말 중요하다
- 값이 커지다 보니 overflow가 잘 일어남. 풀이2의 경우 minheight는 사실 int로 해도 되는데 cost를 계산하는 시점에서 a\*b와 곱하면서 overflow가 나버린다.
	- 이런 문제에서는 메모리가 정말 부족하지 않은 이상 그냥 64bit 써버리는게 속 편할지도 모르겠다.

# 풀이 1
## 결국 풀어야 하는 문제들:
1. a X b 안에 있는 숫자중 가장 작은 숫자는 무엇인가?
2. a X b 안에 있는 숫자를 모두 더하면 무엇인가?
3. cost가 가장 작은 것은 무엇인가?
4. 겹치는지 안겹치는지 어떻게 알 수 있는가?

### a X b 안에 있는 숫자중 가장 작은 숫자를 구하기 위해 min-heap을 사용함
- 먼저 각 row마다 min-heap을 construct하고 각 node가 어디부터 어디까지 범위의 숫자의 min을 의미하는지 기억하게 한다
- 다음과 같이 traverse한다
	- 1. 올라가기
		- 내가 right child면 일단 parent의 parent로 올라간다
		- 내가 left child면 parent의 범위를 본다.
			- 시작 + b가 parent의 범위 오른쪽 끝과 같다면 그냥 right child의 값과 비교하고 min값을 리턴하면 됨
			- 시작 + b가 parent의 범위 오른쪽 끝보다 크다면 parent의 오른쪽 끝까지는 skip 할 수 있는 것이다. parent의 right child와 비교해서 min을 갱신하고 parent의 parent로 올라간다
			- 시작 + b가 parent의 범위 오른쪽 끝보다 작다면 parent의 오른쪽 child로 이동한 다음 아래로 내려간다
	- 2. 내려가기
		- 도착한 지점이 시작+b 면 마지막 값과 비교하고 리턴하면 됨
		- 시작 + b가 parent의 왼쪽 child의 오른쪽 끝과 같다면 그냥 왼쪽 child의 오른쪽 끝과 비교해서 min값을 리턴하면 됨
		- 시작 + b가 parent의 왼쪽 child의 오른쪽 끝보다 크다면 일단 왼쪽 child의 오른쪽 끝과 비교해서 min값을 갱신하고 오른쪽 아래로 내려간다
		- 시작 + b가 parent의 왼쪽 child의 오른쪽 끝보다 작다면 왼쪽 child로 내려간다
- 이렇게 하면 logN만에 시작->시작+b까지의 min을 얻을 수 있음
- 위를 이용해 row에 대해 만들고, 그것을 이용해 column에 대해 만들면 어떤 점을 기점으로 오른쪽으로 b, 아래로 a 안의 min을 구할 수 있음
- 총 시간 복잡도는 NlgN

### a X b 안에 있는 숫자를 모두 더했을때의 값을 구하기 위해서 다음 방법을 사용함
- 처음엔 b개를 모두 더함.
- 그 다음번 부터는 왼쪽 끝을 빼고 오른쪽 끝을 더함
- 이런 방식으로 어떤점을 시작으로 b개를 더한 것을 빠르게 구할 수 있다
- 세로에 대해 반복하면 a X b안에 있는 것을 더한 값을 구할 수 있음
- 대략 N

### cost가 가장 작은것
- (cost,row,column) 데이터 쌍을 cost->row->column priority로 sort 함
- stl sort를 쓴다. NlgN

### 겹치는지 안겹치는지?
- sort한 것에서 하나씩 꺼내서 배치한다
- 배치할 때 마다 해당 area를 표시한다
- 만약 겹치면 사각형의 네 꼭지점중 하나가 반드시 이미 표시되어 있다. 즉 네 꼭지점만 조사해 보면 겹치는게 있는지 없는지 확인할 수 있음


# 풀이 2
### 다른 풀이를 해본 이유
- 풀이1을 다 짜고 보니 300줄. 너무 길다!
- 인터넷을 찾다보니 이런게 나옴: [https://www.programmersought.com/article/8605496453/][https://www.programmersought.com/article/8605496453/]
- 꽤 신박해서 나름대로 분석한 다음 내가 다시한번 짜 봤다

### a X b 안에 있는 숫자중 가장 작은 숫자 구하기
- 글로 설명하기 힘든데, 다음과 같은 DS를 만들면 된다
- ![](/assets/images/codeforces_15D_mink.jpg)
- 당연하지만, K가 8이라면 위로 한단계 더 만들고, 16이상이면 한단계 더 만들고 하는 식이다
- 한단계 위로 올라갈 때 마다 x..x+(2^k)에 대한 min이 x번째 element가 되는 셈
- 이렇게 하면 logK만에 특정 시점부터 K개 안의 min을 구할수 있다
- 내가 사용한 방법보다 훨씬 설명도 짧고 time complexity도 낮고 빠르다. 좋은걸 배웠음.
- 당연하겠지만, max나 다른 것에도 이용해 먹을 수 있을듯?

### a X b 안에 있는 숫자를 모두 더했을때의 값 구하기
- sum\[row\]\[col\] = sum\[row-1\]\[col\] + sum\[row\]\[col-1\] - sum\[row-1\]\[col-1\] + height\[row\]\[col\];
- 한마디로 2D partial sum을 만들면 되는것
- 더 높은 dimension에 대해서도 비슷한 접근이 가능할듯?

### 그 외
- 나머지는 풀이 1과 같았음

### 결과 코드

```cpp
#include <stdio.h>
#include <vector>
#include <algorithm>
#include <cmath>

int height[1001][1001] = {0};
long long sum[1001][1001] = {0};

int mink[1001][21] = {0};
long long minheight[1001][1001] = {0};

typedef struct _abdiff{
	long long cost;
	int row;
	int col;
	friend bool operator<(const struct _abdiff& a, const struct _abdiff& b){
		if(a.cost == b.cost){
			if(a.row == b.row){
				return a.col < b.col;
			}
			return a.row < b.row;
		}
		return a.cost < b.cost;
	}
} abdiff;

abdiff diffrank[(1000*1000)+1];
char emap[1001][1001] = {0};

int main(){
	int n,m,a,b;

	// get input
	scanf("%d %d %d %d",&n,&m,&a,&b);
	// we use this ordering for convenience in computing 2d sum
	for(int row=1; row <= n; ++row){ 
		for(int col=1; col <= m; ++col){
			scanf("%d", &height[row][col]);
		}
	}

	// compute 2d partial sum
	for(int row=1; row<=n; ++row){
		for(int col=1; col<=m; ++col){
			sum[row][col] = sum[row-1][col] + sum[row][col-1] - sum[row-1][col-1] + height[row][col];
			//printf("%lld ", sum[row][col]);
		}
		//printf("\n");
	}

	// build min-k tree for column
	for(int row=1; row<=n; ++row){
		for(int col=1; col<=m; ++col){
			mink[col][0] = height[row][col];
		}
		int k = 1;
		for(;(1<<k) <= b; ++k){
			for(int col=1; col+(1<<k)-1<=m; ++col){
				mink[col][k] = std::min(mink[col][k-1], mink[col + (1<<(k-1))][k-1]);
			}
		}
		k = log(b*1.0)/log(2.0);
		for(int col=1; col<=m-b+1; ++col){
			minheight[row][col] = std::min(mink[col][k], mink[col + b - (1<<k)][k]);
		}
	}

	// build min-k tree for row
	for(int col=1; col<=m-b+1; ++col){
		for(int row=1; row<=n; ++row){
			mink[row][0] = minheight[row][col];
		}
		int k = 1;
		for(;(1<<k) <= a; ++k){
			for(int row=1; row+(1<<k)-1<=n; ++row){
				mink[row][k] = std::min(mink[row][k-1], mink[row + (1<<(k-1))][k-1]);
			}
		}
		k = log(a*1.0)/log(2.0);
		for(int row=1; row<=n-a+1; ++row){
			minheight[row][col] = std::min(mink[row][k], mink[row + a - (1<<k)][k]);
		}
	}

	int diffrank_len = 0;
	for(int row=1; row<=n-a+1; ++row){
		for(int col=1; col<=m-b+1; ++col){
			long long cost = sum[row+a-1][col+b-1] - sum[row+a-1][col-1] - sum[row-1][col+b-1] + sum[row-1][col-1] - (minheight[row][col] * a * b);
			diffrank[diffrank_len] = {cost, row, col};
			++diffrank_len;
		}
	}

	std::sort(diffrank, diffrank+diffrank_len);

	std::vector<long long> rescost;
	std::vector<int> resrow;
	std::vector<int> rescol;

	for(int i=0; i<diffrank_len; ++i){
		int row = diffrank[i].row;
		int col = diffrank[i].col;

		if(!emap[row][col] && !emap[row+a-1][col] && !emap[row][col+b-1] && !emap[row+a-1][col+b-1]){
			rescost.push_back(diffrank[i].cost);
			resrow.push_back(diffrank[i].row);
			rescol.push_back(diffrank[i].col);
			for(int rr = row; rr <= row+a-1; ++rr){
				for(int cc=col; cc <= col+b-1; ++cc){
					emap[rr][cc] = 1;
				}
			}
		}
	}

	printf("%lu\n", rescost.size());
	for(int i=0; i<rescost.size(); ++i){
		printf("%d %d %lld\n", resrow[i], rescol[i], rescost[i]);
	}


	return 0;
}
```
