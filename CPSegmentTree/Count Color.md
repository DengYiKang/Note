# Count Color

## POJ 2777

Chosen Problem Solving and Program design as an optional course, you are required to solve all kinds of problems. Here, we get a new problem. 

There is a very long board with length $L$ centimeter, $L$ is a positive integer, so we can evenly divide the board into $L$ segments, and they are labeled by $1, 2, ... L$ from left to right, each is 1 centimeter long. Now we have to color the board - one segment with only one color. We can do following two operations on the board: 

+ $C A\,B\,C$ Color the board from segment $A$ to segment $B$ with color $C$. 
+ $P\,A\,B$ Output the number of different colors painted between segment $A$ and segment $B$ (including). 

In our daily life, we have very few words to describe a color (red, green, blue, yellow…), so you may assume that the total number of different colors $T$ is very small. To make it simple, we express the names of colors as $color 1, color 2, ... color T$. At the beginning, the board was painted in $color 1$. Now the rest of problem is left to your.   

### Input

First line of input contains $L\, (1 <= L <= 100000)$, $T (1 <= T <= 30)$ and $O (1 <= O <= 100000)$. Here $O$ denotes the number of operations. Following $O$ lines, each contains $P\, A\, B\, C$ or $P\, A\, B$ (here $A, B, C$ are integers, and $A$ may be larger than $B$) as an operation defined previously.

### Output

Output results of the output operation in order, each line contains a number.

### Sample Input

```c++
2 2 4
C 1 1 2
P 1 2
C 2 2 2
P 1 2
```

### Sample Output

```c++
2
1
```

### Analysis

+ We can use **int** to store the information of any segment, since the total number of colors is 30. We can use bit operations to reduce the time cost.
+ Each vertex store the **int** val which described above using **Segment Tree**.

### Code

```c++
#include<iostream>
#include<cstdio>
#include<algorithm>

using namespace std;

const int maxn=1e5+5;
int t[maxn<<2];
int lazy[maxn<<2];

void push(int v){
	if(lazy[v]==0) return;
	lazy[v<<1]=lazy[v<<1|1]=lazy[v];
	t[v<<1]=t[v<<1|1]=1<<(lazy[v]-1);
	lazy[v]=0;
}

void build(int v, int tl, int tr){
	if(tl==tr) t[v]=1;
	else{
		int mid=(tl+tr)>>1;
		build(v<<1, tl, mid);
		build(v<<1|1, mid+1, tr);
	}
	lazy[v]=0;
	t[v]=1;
}
void update(int v, int tl, int tr, int l, int r, int new_val){
	if(l>r) return;
	if(tl==l&&tr==r) {
		t[v]=1<<(new_val-1);
		lazy[v]=new_val;
		return;
	}
	int mid=(tl+tr)>>1;
	push(v);
	update(v<<1, tl, mid, l, min(r, mid), new_val);
	update(v<<1|1, mid+1, tr, max(l, mid+1), r, new_val);
	t[v]=t[v<<1]|t[v<<1|1];
}
int query(int v, int tl, int tr, int l, int r){
	if(l>r) return 0;
	if(tl==l&&tr==r) return t[v];
	int mid=(tl+tr)>>1;
	push(v);
	return query(v<<1, tl, mid, l, min(r, mid))
	|query(v<<1|1, mid+1, tr, max(l, mid+1), r);
}
int main(){
	int L, T, O;
	scanf("%d%d%d", &L, &T, &O);
	build(1, 1, L);
	while(O--){
		getchar();
		char ch=getchar();
		int op1, op2, val;
		scanf("%d%d", &op1, &op2);
		if(op1>op2){
			op1^=op2;
			op2^=op1;
			op1^=op2;
		}
		if(ch=='C') {
			scanf("%d", &val);
			update(1, 1, L, op1, op2, val);
		}
		else {
			int ans=0, num=query(1, 1, L, op1, op2);
			for(int i=0; i<T; i++) 
				if(num&(1<<i)) ans++;
			printf("%d\n", ans);
		}

	}
	return 0;
}

```

