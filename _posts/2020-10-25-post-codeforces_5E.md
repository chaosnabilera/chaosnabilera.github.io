---
title: "codeforces.com - 5E"
date: 2020-10-25
categories:
  - Problem Solving
tags:
  - codeforces.com
  - problem solving
---

[codeforces 5E]: http://codeforces.com/problemset/problem/5/E
[https://www.programmersought.com/article/42685010947/]: https://www.programmersought.com/article/42685010947/
[https://codeforces.com/blog/entry/213]:https://codeforces.com/blog/entry/213

[codeforces.com 5E Bindian Signalizing][codeforces 5E] 문제풀이

# 문제
- 원 위에 n개의 언덕이 있고 (3 <= n <= 10^6) 각각 1 <= h <= 10^9 의 높이를 가진다
- 언덕 A와 언덕 B 사이에 A이나 B보다 더 높은 언덕이 없다면 A와 B는 서로 볼 수 있다
- 서로 볼 수 있는 언덕 쌍의 총 개수를 구해라

# 풀이

- 생각으로는 간단한데 글로 풀어쓰려니까 생각 이상으로 길다....

## 문제의 의미
- 원 위에 언덕이 있는 것이므로 한 언덕에서 왼쪽으로 볼수도 있고 오른쪽으로 볼수도 있음
- 순서는 중요치 않다. 즉 A,B = B,A
- h(X)를 X의 높이라고 했을 때 세는 것의 중복을 막기 위해서는 언제나 h(A) < h(B) 인 pair만 세고, h(A)=h(B)인 pair에 대해서는 따로 세면 된다
- A나 B보다 **높은** 언덕이 없으면 되는 거니까 A나 B와 같은 높이의 언덕이 중간에 있더라도 A와 B는 서로 마주 볼 수 있음
- 가장 높은 언덕보다 낮은 언덕은 왼쪽으로 보든 오른쪽으로 보든 가장 높은 언덕에 시야가 걸리게 되어 있음
- 따라서 가장 높은 언덕의 높이를 H라고 한다면 원의 형태에서 H, a, b, c, ..., z, H 와 같은 형태로 문제를 변형하고 풀 수 있음 (그 사이에 H와 같은 높이의 언덕이 있다고 하더라도 별 상관 없음)
- 중복을 제외하고 세려면 다음과 같이 세면 됨
	- 자신을 X라고 하자
	- 이웃한 hill은 따로 센다
	- 자신의 오른쪽에 있는 것 중에 자신보다 높은 것 중 가장 가까운 것을 센다. 이것을 Y라고 하자.
	- X와 Y사이에 있는 hill중 X와 같은 높이의 hill 들을 센다 (Y 전에 있고 X와 높이가 같은 hill들을 제외한 나머지는 위의 정의에 의해 무조건 높이가 X와 같으므로 pair가 될 수 있음)
		- 이건 자신의 왼쪽으로만 세든지 자신의 오른쪽으로만 세든지 한쪽 방향으로만 세야함. 그렇지 않으면 중복으로 세어진다
		- 다시 말해, 한쪽 방향으로는 자신보다 높은것과의 pair, 자신과 같은 높이와의 pair 둘다 센다면 다른 방향으로는 자신보다 높은것과의 pair만 세어야 한다

## 결국 풀기 위해 필요한 것?
- 주어진 input에서 가장 높은 것의 index를 hh라고 할 때, 그것을 기준으로 원을 펴서 양 끝에 가장 높은 hill 이 오도록 한 chain으로의 변환.
- chain[i] = 위의 변환한 chain에서 i번째 element의 높이
- L_hi[i] = i번째 element에서 L->R 방향으로 봤을때 가장 처음 만나는 i보다 높은 hill의 index
- L_numsame[i] = i번째 element에서 L->R 방향으로 봤을때 가장 처음 만나는 i보다 높은 hill의 index
- R_hi[i] = L_hi의 반대방향(R->L) 버전

## 위를 가지고 답을 어떻게 알 수 있나?
- L[i] = chain의 i번째에서 L->R 방향으로 셀 수 있는 pair의 개수
	- L[i] = 1 + L_numsame[i] + 1
		- 처음의 1은 이웃과의 pair (무조건 생김)
		- 마지막 1은 L_hi[i] 와 생기는 pair
			- L_hi[i]가 i+1이라면 생기지 않음. 이에 대한 예외 처리 있어야 함
		- L_numsame[i]는 i와 L_hi[i] 사이에 있는 i와 같은 높이의 hill 개수
			- 바로 이웃이 같은 높이라면 -1 해야 함
			- i의 높이가 가장 높은 hill의 높이와 같다면 예외 처리 해야 함
- R[i] = L[i]의 반대방향 버전
	- R[i] = 1 (이웃은 이미 L[i]에서 셌고, 자신보다 높은 것과의 pair만 세니까)
		- 단, L_hi[i]가 오른쪽 끝이었다면 chain은 결국 원의 하나를 기준으로 잡고 편것이므로 오른쪽끝 = 왼쪽끝이라 이미 L[i]에서 셌음. 즉 중복이니 빼야 함
		- 자신의 높이가 가장 큰 높이와 같다면 L[i]에서 이미 셌으니 0

## L_hi, L_numsame, R_hi를 나는 어떻게 구했는가?
- chain은 trivial함
- L_hi, R_hi는 결국 어떤 한 방향으로 자신보다 더 큰 element를 찾아내야 한다.
	- 처음엔 그냥 무식하게 풀음. 이렇게 하면 input이 작을때나 높이가 적절히 섞여 있을때는 의외로 괜찮다. 그러나 worst-case scenario에서는 바로 N^2가 되므로 timeout이 나버린다
		- worst-case scenario는 L->R방향으로 보는 경우 H, k, k-1, k-2, k-3, ... , H가 되는 형태. 하나씩 보는 식으로 짜면 자연스럽게 N^2이 됨
	- 이런저런 고민을 하다 binary tree로 해결. max heap을 이용하면 한쪽 방향으로 자신보다 더 큰 것중 자신과 가장 가까운 것을 찾을 수 있음. 한번에 복잡도는 logN이므로 NlogN으로 풀 수 있다
- L_numsame
	- 처음엔 L_hi와 함께 무식하게 셌으나 L_hi를 무식하게 풀면 timeout이 남에 따라 포기
	- i번째의 높이를 h(i)라고 했을때, (h(i),i) pair의 array를 만들고 이를 h(i)에 대해서는 내림차순으로, i에 대해서는 오름차순으로 정렬함
	- 일단 L_hi를 구한 다음, 위의 정렬된 array를 이용해서 i와 같은 h를 가진 것 중에 L_hi보다 전에 나오는 것들만 세는 식으로 해결

## 소스 코드
```cpp
#include <iostream>
#include <vector>
#include <cstring>
#include <algorithm>
#include <utility>

typedef struct _GNode{
	int val;
	int idx;
	struct _GNode* parent;
	struct _GNode* left;
	struct _GNode* right;
	char mode;
} GNode;

void printIntArray(const int* array, const int array_length) {
	for (int i = 0; i < array_length; ++i) {
		std::cout << array[i] << " ";
	}
	std::cout << std::endl;
}

void makeTree(const int* chain, const int chain_length, GNode*& tree_output, int& tree_size_output){
	int lastlvsize = 0;
	int base_idx = 0;
	int curi, li, ri = 0;

	int curlvsize = chain_length;
	int totalsize = curlvsize;

	while(curlvsize > 1){
		curlvsize = (curlvsize & 1) ? (curlvsize / 2) + 1 : (curlvsize / 2);
		totalsize += curlvsize;
	}

	GNode* tree = new GNode[totalsize];
	memset(tree, 0, sizeof(GNode)*totalsize);

	for(int i=0; i<chain_length; ++i){
		tree[i].val = chain[i];
		tree[i].idx = i; // only leaf
	}

	lastlvsize = chain_length;
	base_idx = lastlvsize;
	while(lastlvsize > 1){
		if(lastlvsize & 1){
			curlvsize = (lastlvsize / 2) + 1;

			curi = base_idx;
			li = (base_idx - lastlvsize);
			tree[curi].left = &(tree[li]);
			tree[curi].left->parent = &(tree[curi]);
			tree[curi].left->mode = 'L';
			tree[curi].val = tree[curi].left->val;

			//std::cout << curi << " " << li << " " << -1 << std::endl;

			for(int i=1; i<curlvsize; ++i){
				li = base_idx - lastlvsize + ((i-1)*2) + 1;
				ri = li + 1;
				curi = base_idx+i;

				//std::cout << curi << " " << li << " " << ri << std::endl;

				tree[curi].left = &(tree[li]);
				tree[curi].left->parent = &(tree[curi]);
				tree[curi].left->mode = 'L';

				tree[curi].right = &(tree[ri]);
				tree[curi].right->parent = &(tree[curi]);
				tree[curi].right->mode = 'R';

				tree[curi].val = ( tree[curi].left->val > tree[curi].right->val ) ? tree[curi].left->val : tree[curi].right->val;
			}
		}
		else{
			curlvsize = (lastlvsize / 2);

			for(int i=0; i<curlvsize; ++i){
				li = base_idx - lastlvsize + (i*2);
				ri = li + 1;
				curi = base_idx+i;

				//std::cout << curi << " " << li << " " << ri << std::endl;

				tree[curi].left = &(tree[li]);
				tree[curi].left->parent = &(tree[curi]);
				tree[curi].left->mode = 'L';

				tree[curi].right = &(tree[ri]);
				tree[curi].right->parent = &(tree[curi]);
				tree[curi].right->mode = 'R';

				tree[curi].val = ( tree[curi].left->val > tree[curi].right->val ) ? tree[curi].left->val : tree[curi].right->val;
			}
		}
		
		lastlvsize = curlvsize;
		base_idx += curlvsize;
	}

	tree_output = tree;
	tree_size_output = totalsize;
}


void printTree(GNode* tree, int tree_size){
	int base_idx = tree_size;
	int curlvsize = 1;
	int nextlvsize;

	while(base_idx > 0){
		for(int j = 0; j < curlvsize; ++j){
			std::cout << tree[base_idx - curlvsize + j].val << "(" << tree[base_idx - curlvsize + j].mode << ") ";
		}
		std::cout << std::endl;

		if(tree[base_idx - curlvsize].right == NULL){
			nextlvsize = (curlvsize*2)-1;
		}
		else{
			nextlvsize = (curlvsize*2);
		}

		base_idx -= curlvsize;
		curlvsize = nextlvsize;
	}
}

bool comparePair(const std::pair<int,int>& a, const std::pair<int,int>& b){
	if(a.first == b.first){
		return a.second < b.second;
	}
	return a.first > b.first;
}

void printStdPairVector(std::vector<std::pair<int, int>>& vec) {
	for(int i=0; i< vec.size(); ++i){
		std::cout << "( " << vec[i].first << ", " << vec[i].second << ")";
	}
	std::cout << std::endl;
}

void computeSameHeight(const int* chain, const int chain_length, const int* L_hi, int*& L_numsame){
	std::vector<std::pair<int,int>> carr;
	for(int i=0; i<chain_length; ++i){
		carr.push_back(std::pair<int,int>(chain[i], i));
	}

	std::sort(carr.begin(), carr.end(), comparePair);
	// printStdPairVector(carr);

	for(int i=0; i<chain_length; ++i){
		L_numsame[i] = 0;
	}

	char* lns_visited = new char[chain_length];
	memset(lns_visited, 0, sizeof(char)*chain_length);
	lns_visited[0] = 1;
	
	int maxh = carr[0].first;
	for(int i=1; i<carr.size(); ){
		int start = i;
		int end = i;
		int numsame = 0;

		if (lns_visited[carr[i].second]) {
			++i;
			continue;
		}

		while((i < carr.size()-1) && (carr[i].first == carr[i+1].first)){
			++i;
		}
		end = i;

		// at this point [start,end] are same numbers

		numsame = end - start;
		if(numsame > 0){
			if(carr[start].first == maxh){ // Simple case
				for(int j = 0; j <= numsame-1; ++j){
					L_numsame[carr[start+j].second] = numsame-1-j;
					lns_visited[carr[start + j].second] = 1;
				}
			}
			else{ // Complicated case
				for (int s = 0; s <= numsame;) {
					int si = carr[start + s].second;
					L_numsame[si] = 0;
					lns_visited[si] = 1;
					for (int n = 1; s + n <= numsame; ++n) {
						int s_p_ni = carr[start + s + n].second;
						if (s_p_ni < L_hi[si]) {
							L_numsame[si]++;
						}
						else {
							break;
						}
					}

					int ns = s + 1;
					for (int n = 1; s + n <= numsame; ++n) {
						int s_p_ni = carr[start + s + n].second;
						if (s_p_ni < L_hi[si]) {
							ns++;
							L_numsame[s_p_ni] = L_numsame[si] - n;
							lns_visited[s_p_ni] = 1;
						}
						else {
							break;
						}
					}

					s = ns;
				}
			}
			i = end + 1;
		}
		else{
			L_numsame[carr[i].second] = 0;
			lns_visited[carr[i].second] = 1;
			++i;
		}
	}

	/*
	for(int i=0; i<chain_length; ++i){
		std::cout << L_numsame[i] << " "; 
	}
	std::cout << std::endl;
	*/
	
}

int traverse_LtoR(const GNode* tree, int start) {
	const GNode* curNode = &(tree[start]);
	int start_val = tree[start].val;
	const GNode* curParent = curNode->parent;

	// Go up
	while (1) {
		// If curNode is right child, just go up
		if (curNode->mode == 'R') {
			curNode = curParent;
			curParent = curNode->parent;
			continue;
		}
		else {
			if ((curParent->val > start_val) && (curParent->right != NULL) && (curParent->right->val > start_val)) {
				curNode = curParent->right;
				break;
			}
			else {
				curNode = curParent;
				curParent = curNode->parent;
			}
		}
	}

	while (1) {
		if ((curNode->left) == NULL && (curNode->right == NULL)) {
			break;
		}

		if ((curNode->left != NULL) && (curNode->left->val > start_val)) {
			curNode = curNode->left;
		}
		else {
			curNode = curNode->right;
		}
	}

	return curNode->idx;
}

int traverse_RtoL(const GNode* tree, int start) {
	const GNode* curNode = &(tree[start]);
	int start_val = tree[start].val;
	const GNode* curParent = curNode->parent;

	// Go up
	while (1) {
		// If curNode is left child, just go up
		if (curNode->mode == 'L') {
			curNode = curParent;
			curParent = curNode->parent;
			continue;
		}
		else {
			if ((curParent->val > start_val) && (curParent->left != NULL) && (curParent->left->val > start_val)) {
				curNode = curParent->left;
				break;
			}
			else {
				curNode = curParent;
				curParent = curNode->parent;
			}
		}
	}

	while (1) {
		if ((curNode->left) == NULL && (curNode->right == NULL)) {
			break;
		}

		if ((curNode->right != NULL) && (curNode->right->val > start_val)) {
			curNode = curNode->right;
		}
		else {
			curNode = curNode->left;
		}
	}

	return curNode->idx;
}

void compute_hi(const GNode* tree_LtoR, const GNode* tree_RtoL, const int* chain, const int chain_length, const int maxh, int*& L_hi, int*& R_hi) {
	for (int i = 1; i < chain_length -1; ++i) {
		if (chain[i] == maxh) {
			L_hi[i] = chain_length - 1;
		}
		else {
			L_hi[i] = traverse_LtoR(tree_LtoR, i-1) + 1;
		}
	}

	for (int i = chain_length - 2; i > 0; --i) {
		if (chain[i] == maxh) {
			R_hi[i] = 0;
		}
		else {
			R_hi[i] = traverse_RtoL(tree_RtoL, i);
		}
	}
}

int main() {
	std::ios_base::sync_with_stdio(false);
	
	int n = 0;
	unsigned long long lln = 0;
	int* hill = NULL;
	int* chain = NULL;
	int chain_length = 0;
	int maxh = 0;
	int maxhi = 0;
	bool allsame = false;
	GNode* tree_LtoR = NULL;
	GNode* tree_RtoL = NULL;
	int tree_size_LtoR = 0;
	int tree_size_RtoL = 0;
	int* L = NULL;
	int* L_hi = NULL;
	int* L_numsame = NULL;
	int* R = NULL;
	int* R_hi = NULL;


	std::cin >> n;

	hill = new int[n];
	chain = new int[n + 1];
	chain_length = n + 1;

	L = new int[chain_length]();
	L_hi = new int[chain_length]();
	L_numsame = new int[chain_length]();
	R = new int[chain_length]();
	R_hi = new int[chain_length]();

	for (int i = 0; i < n; ++i) {
		std::cin >> hill[i];
	}

	maxh = hill[0];
	maxhi = 0;
	allsame = true;

	/* stupid...
	for (int i = 1; i < n; ++i) {
		if (hill[i] > maxh) {
			maxh = hill[i];
			maxhi = i;
		}
		if (hill[i] != maxh) {
			allsame = false;
		}
	}
	*/

	for (int i = 1; i < n; ++i) {
		if (hill[i] > maxh) {
			maxh = hill[i];
			maxhi = i;
		}
		if (hill[i] != hill[0]) {
			allsame = false;
		}
	}

	if (allsame) {
		lln = n;
		std::cout << (lln*(lln - 1)) / 2 << std::endl;
		return 0;
	}

	chain[0] = maxh;
	for (int i = 1; i < chain_length - maxhi; ++i) {
		chain[i] = hill[maxhi + i];
	}
	for (int i = 0; i <= maxhi; ++i) {
		chain[chain_length - maxhi + i - 1] = hill[i];
	}

	/*
	for (int i = 0; i < chain_length; ++i) {
		std::cout << chain[i] << " ";
	}
	std::cout << std::endl;
	*/

	makeTree(&chain[1], chain_length-1, tree_LtoR, tree_size_LtoR);
	//printTree(tree_LtoR, tree_size_LtoR);

	makeTree(&chain[0], chain_length - 1, tree_RtoL, tree_size_RtoL);
	//printTree(tree_RtoL, tree_size_RtoL);

	compute_hi(tree_LtoR, tree_RtoL, chain, chain_length, maxh, L_hi, R_hi);

	//printf("L_hi\n");
	//printIntArray(L_hi, chain_length);
	//printf("R_hi\n");
	//printIntArray(R_hi, chain_length);

	computeSameHeight(chain, chain_length, L_hi, L_numsame);
	//printf("L_numsame\n");
	//printIntArray(L_numsame, chain_length);

	for (int i = 1; i < chain_length - 2; ++i) {
		L[i] = 1 + L_numsame[i] + 1;

		if ((L_numsame[i] > 0) && (chain[i] == chain[i + 1])) {
			L[i] -= 1;
		}
		if ((i + 1 == L_hi[i]) && (chain[i] < chain[L_hi[i]])) {
			L[i] -= 1;
		}
	}
	L[chain_length - 2] = 1;

	
	for (int i = chain_length - 2; i > 0; --i) {
		R[i] = 1;

		if (chain[i] == maxh) {
			R[i] = 0;
		}
		else if ((R_hi[i] == 0) && (L_hi[i] == chain_length - 1)) {
			R[i] = 0;
		}
		else if ((i > 1) && (R_hi[i] == i - 1)) {
			R[i] = 0;
		}
	}

	//printf("L\n");
	//printIntArray(L, chain_length);
	//printf("R\n");
	//printIntArray(R, chain_length);

	long long sum = 0;

	for (int i = 0; i < chain_length; ++i) {
		sum += L[i];
		sum += R[i];
	}

	std::cout << sum << std::endl;

	return 0;
}
```

## 반성
- 이 끔찍하게 긴 코드를 보면 자명하지만, 이 문제는 이렇게 풀라고 만든 문제가 아니다...
- tree traverse 하는게 들어가니까 쓸데 없이 복잡해져서 버그가 많았다
- L_numsame 구하는 방식도 마찬가지.
- 이렇게 복잡해 지고 나니 edge 케이스 처리가 어려워서 디버깅 하는게 너무 힘들었음 ㅠ
- 19번의 submission끝에 성공했음...

## 모범답안
- 출처: [https://www.programmersought.com/article/42685010947/][https://www.programmersought.com/article/42685010947/]

```c

#include <iostream>
#include <string.h>
#include <algorithm>
#define STD std::ios::sync_with_stdio(false);cin.tie(0);cout.tie(0)
const int maxn=1e6+10;
int a[maxn],b[maxn],l[maxn],r[maxn],s[maxn];
using namespace std;
int main()
{
	STD;
	int n,maxn,po;
	cin>>n;
	for (int i=0;i<n;i++)
		cin>>a[i]; // a가 input array임
	maxn=a[0]; //max값
	po=0; //max의 position
	for (int i=1;i<n;i++)
	{
		if (a[i]>maxn)
		{
			maxn=a[i];
			po=i;
		}
	}
	// b가 chain. 신박한 방법이다....
	for (int i=0;i<n;i++)
		b[i]=a[(po+i)%n];
	b[n]=b[0]; // chain 마지막에 최대값

	s[n]=0; // ????
	for (int i=n-1;i>=0;i--)
	{
		r[i]=i+1;
		while (r[i]<n&&b[i]>b[r[i]])
			r[i]=r[r[i]];
		while (r[i]<n&&b[i]==b[r[i]])
		{
			s[i]=s[r[i]]+1;
			r[i]=r[r[i]];	
		}	
	}	
	l[0]=0;
	for (int i=1;i<n;i++)
	{
		l[i]=i-1;
		while (l[i]>0&&b[i]>=b[l[i]])
			l[i]=l[l[i]];
	}
	long long ans=0;
	for (int i=0;i<n;i++)
	{
		ans+=s[i];
		if (b[i]<b[0])
		{
			if (l[i]==0&&r[i]==n)
				ans+=1;
			else
				ans+=2;
		}
	}
	cout<<ans<<endl;
	return 0;
}
```

## 모범답안 해설
- Editorial이 있었는데 못찾았네... 결국 하는 얘기는 아래와 비슷함.
	- [https://codeforces.com/blog/entry/213][https://codeforces.com/blog/entry/213]
- chain만드는게 심플하고 (내 입장에서는) 신박하다. 그래 %를 이용하면 저렇게 쉽게 표현할 수 있군...
- L_hi를 구하려면 오른쪽 끝에서 부터 구하면 되는 거였다.
	- N-1번째는 당연히 N번째랑만 pair가 될 수 있음
	- 내 오른쪽 옆에 있는게 나와 같다면 그것의 L_hi는(정확히는 그 chain을 따라가면) 나의 L_hi임
	- 내 오른쪽 옆에 있는게 나보다 작다면 그것의 L_hi는 나의 L_hi이든지, 아니면 L_hi[L_hi]가 나의 L_hi이든지, 여튼 저런 식으로 따라가면 나의 L_hi가 나옴
	- 즉 L_hi는 linear하게 구할 수 있다
		- H, k, 1, 2, ...., k-1, H 인 경우에는 k가 N번 jump하긴 해야 하는데, 그 전엔 전부 1번만 보면 되니까 결국 N...
		- H, k, k-1, ..., 2, 1, 2, 3, ... k, H인 경우에도 결국 오른쪽끝에서 시작하면 첫 반은 한번에 되고 두번째 반쪽 부터는 2번만에 된다. 그러니 결국 O(N)
- L_numsame은 L_hi를 구하는 도중에 자연스럽게 구해진다
- R_hi도 위와 비슷하게 구하면 됨