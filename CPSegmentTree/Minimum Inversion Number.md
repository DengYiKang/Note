# Minimum Inversion Number

## HDU 1394

The inversion number of a given number sequence $a_1, a_2, ..., a_n$ is the number of pairs $(a_i, a_j)$ that satisfy $i < j$ and $a_i > a_j$. 

For a given sequence of numbers $a_1, a_2, ..., a_n$, if we move the first $m >= 0$ numbers to the end of the sequence, we will obtain another sequence. There are totally n such sequences as the following: 

$a_1, a_2, ..., a_{n-1}, a_n$ (where $m = 0$ - the initial sequence) 
$a_2, a_3, ..., a_n, a_1$ (where $m = 1$) 
$a_3, a_4, ..., a_n, a_1, a_2$ (where $m = 2$) 
... 
$a_n, a_1, a_2, ..., a_{n-1}$ (where $m = n-1$) 

You are asked to write a program to find the minimum inversion number out of the above sequences.   

### Input

The input consists of a number of test cases. Each case consists of two lines: the first line contains a positive integer n (n <= 5000); the next line contains a permutation of the n integers from 0 to n-1. 

### Output

For each case, output the minimum inversion number on a single line.

### Sample input

```c++
10
1 3 6 9 0 8 5 7 4 2
```

### Sample output

```c++
16
```

### Analysis

+ Inversion pairs are **symmetric**. That is, if $(a_i, a_j)$ is an inversion pair, if you count it from the side of $a_i$, then you can not count it from the side of $a_j$, since  it's **repetitive**.  So we can use the property of input (**Sequentiality**) to compute the initial inversion number. We sign each number which have been inputed, for a coming number $a[i]$ , we compute the inversion number of $a[i]$ from the previous input and add it to **sum**. We can use **Segment Tree** to complete it.
+ For each operation moving the first number, assuming that $a[i]$ is the $ith$ number in current status, and $sum$ is the initial inversion number of the current status, then $sum'=sum+n-2*a[i]-1$ , where $n$ is the size of the array.

### Code

```c++
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<algorithm>
#include<cmath>
using namespace std;

const int maxn=5e3;
int t[maxn<<2];

void build(int v, int tl, int tr){
	if(tl==tr) t[v]=0;
	else{
		int mid=(tl+tr)>>1;
		build(v<<1, tl, mid);
		build(v<<1|1, mid+1, tr);
		t[v]=0;
	}
}
void update(int v, int tl, int tr, int pos){
	if(tl==tr) t[v]=1;
	else{
		int mid=(tl+tr)>>1;
		if(pos<=mid) update(v<<1, tl, mid, pos);
		else update(v<<1|1, mid+1, tr, pos);
		t[v]=t[v<<1]+t[v<<1|1];
	}
}
int query(int v, int tl, int tr, int l, int r){
	if(l>r) return 0;
	if(tl==l&&tr==r) return t[v];
	int mid=(tl+tr)>>1;
	return query(v<<1, tl, mid, l, min(r, mid))
	+query(v<<1|1, mid+1, tr, max(l, mid+1), r);
}
int main(){
	int n;
	while(scanf("%d", &n)!=EOF){
		int ans=0;
		int sum=0;
		int* a=new int[n+1];
		build(1, 1, n);
        //compute the initial inversion number
		for(int i=1; i<n+1; i++){
			scanf("%d", a+i);
			sum+=query(1, 1, n, a[i]+1, n);
			update(1, 1, n, a[i]+1);
		}
		ans=sum;
		for(int i=1; i<n+1; i++){
			sum+=n-2*a[i]-1;
			ans=min(ans, sum);
		}
		printf("%d\n", ans);
	}
	return 0;
}

```

