# Hotel

## POJ 3667

The cows are journeying north to Thunder Bay in Canada to gain cultural enrichment and enjoy a vacation on the sunny shores of Lake Superior. Bessie, ever the competent travel agent, has named the Bull moose Hotel on famed Cumberland Street as their vacation residence. This immense hotel has $N (1 ≤ N ≤ 50,000) $ rooms all located on the same side of an extremely long hallway ( all the better to see the lake, of course ).

The cows and other visitors arrive in groups of size $Di (1 ≤ Di ≤ N)$ and approach the front desk to check in. Each group $i$ requests a set of $Di$ contiguous rooms from Canmuu, the moose staffing the counter. He assigns them some set of consecutive room numbers $r..r+Di-1$ if they are available or, if no contiguous set of rooms is available, politely suggests alternate lodging. Canmuu always chooses the value of $r$ to be the smallest possible.

Visitors also depart the hotel from groups of contiguous rooms. Checkout $i$ has the parameters $Xi$ and $Di$ which specify the vacating of rooms $Xi ..Xi +Di-1\, (1 ≤ Xi ≤ N-Di+1)$. Some (or all) of those rooms might be empty before the checkout.

Your job is to assist Canmuu by processing $M (1 ≤ M < 50,000) $check in/checkout requests. The hotel is initially unoccupied.

### Input

Line $1$: Two space-separated integers: $N$ and $M$

Lines $2..M+1$: Line $i+1$ contains request expressed as one of two possible formats:

* Two space separated integers representing a check-in request: $1$ and $Di$
* Three space-separated integers representing a check-out: $2$, $Xi$, and $Di$

### Output

Lines: For each check-in request, output a single line with a single integer *r*, the first room in the contiguous sequence of rooms to be occupied. If the request cannot be satisfied, output 0.

### Sample Input

```c++
10 6
1 3
1 3
1 3
1 3
2 5 5
1 6
```

### Sample Output

```c++
1
4
7
0
5
```

### Analysis

+ Store the length of continuous prefix, suffix and the whole segment in Segment Tree.
+ The interesting point is at function **query**.

### Code

```c++
#include<iostream>
#include<cstdio>
#include<algorithm>
#include<cmath>

using namespace std;

const int maxn=5e4;
int t[maxn<<2];
int lazy[maxn<<2];
int pref[maxn<<2];
int suff[maxn<<2];

void up(int v, int tl, int mid, int tr){
	if(t[v<<1]==mid-tl+1) pref[v]=t[v<<1]+pref[v<<1|1];
	else pref[v]=pref[v<<1];
	if(t[v<<1|1]==tr-mid) suff[v]=suff[v<<1]+t[v<<1|1];
	else suff[v]=suff[v<<1|1];
	t[v]=max(max(max(pref[v], suff[v]), t[v<<1])
             ,max(suff[v<<1]+pref[v<<1|1],t[v<<1|1]));
}
void down(int v, int tl, int mid, int tr){
	if(lazy[v]==-1) return;
	lazy[v<<1]=lazy[v<<1|1]=lazy[v];
	t[v<<1]=pref[v<<1]=suff[v<<1]=lazy[v]*(mid-tl+1);
	t[v<<1|1]=pref[v<<1|1]=suff[v<<1|1]=lazy[v]*(tr-mid);
	lazy[v]=-1;
}

void build(int v, int tl, int tr){
	if(tl==tr) pref[v]=suff[v]=t[v]=1;
	else{
		int mid=tl+tr>>1;
		build(v<<1, tl, mid);
		build(v<<1|1, mid+1, tr);
		up(v, tl, mid, tr);
	}
	lazy[v]=-1;
}
/*
	important
*/
int query(int v, int tl, int tr, int wide){
	if(tl==tr) return tl;
	int mid=tl+tr>>1;
	down(v, tl, mid, tr);
	if(t[v<<1]>=wide) return query(v<<1, tl, mid, wide);
	if(suff[v<<1]+pref[v<<1|1]>=wide) return mid-suff[v<<1]+1;
	return query(v<<1|1, mid+1, tr, wide);
}
void update(int v, int tl, int tr, int l, int r, int val){
	if(l>r) return;
	if(tl==l&&tr==r){
		pref[v]=suff[v]=t[v]=val*(tr-tl+1);
		lazy[v]=val;
		return;
	}
	int mid=tl+tr>>1;
	down(v, tl, mid, tr);
	update(v<<1, tl, mid, l, min(r, mid), val);
	update(v<<1|1, mid+1, tr, max(l, mid+1), r, val);
	up(v, tl, mid, tr);
}
int main(){
	int n, m;
	while(scanf("%d%d", &n, &m)!=EOF){
		build(1, 1, n);
		for(int i=0; i<m; i++){
			int type, w, l;
			scanf("%d", &type);
			if(type==1){
				scanf("%d", &w);
				int ans=t[1]>=w?query(1, 1, n, w):0;
				printf("%d\n", ans);
				if(ans) update(1, 1, n, ans, ans+w-1, 0);
			}
			else {
				scanf("%d%d", &l, &w);
				update(1, 1, n, l, l+w-1, 1);
			}
		}
	}
	return 0;
}
```

