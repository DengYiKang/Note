# Mayor's posters(RE!)

## POJ 2528

The citizens of Bytetown, AB, could not stand that the candidates in the mayoral election campaign have been placing their electoral posters at all places at their whim. The city council has finally decided to build an electoral wall for placing the posters and introduce the following rules: 

- Every candidate can place exactly one poster on the wall. 
- All posters are of the same height equal to the height of the wall; the width of a poster can be any integer number of bytes (byte is the unit of length in Bytetown). 
- The wall is divided into segments and the width of each segment is one byte. 
- Each poster must completely cover a contiguous number of wall segments.

They have built a wall $10000000$ bytes long (such that there is enough place for all candidates). When the electoral campaign was restarted, the candidates were placing their posters on the wall and their posters differed widely in width. Moreover, the candidates started placing their posters on wall segments already occupied by other posters. Everyone in Bytetown was curious whose posters will be visible (entirely or in part) on the last day before elections.

 Your task is to find the number of visible posters when all the posters are placed given the information about posters' size, their place and order of placement on the electoral wall. 

### Input

The first line of input contains a number $c$ giving the number of cases that follow. The first line of data for a single case contains number $1 <= n <= 10000$. The subsequent $n$ lines describe the posters in the order in which they were placed. The $i\,th$ line among the $n$ lines contains two integer numbers $li$ and $ri$ which are the number of the wall segment occupied by the left end and the right end of the $i\,th$ poster, respectively. We know that for each $1 <= i <= n, 1 <= l i <= ri <= 10000000$. After the $i\,th$ poster is placed, it entirely covers all wall segments numbered $li, li+1 ,... , ri$.

### Output

For each input data set print the number of visible posters after all the posters are placed. 

### Sample Input

```c++
1
5
1 4
2 6
8 10
3 4
7 10
```

### Sample Output

```c++
4
```

### Analysis

+ **Discretization**

  ```c++
  sort(vec, vec+(len<<1));
  len=unique(vec, vec+(len<<1))-vec;
  for(int i=len-2; i>=0; i--){
  	if(vec[i+1]-vec[i]>1) {
  		vec[len++]=vec[i+1]+vec[i]>>1;
  	}
  }
  sort(vec, vec+len);
  len=unique(vec, vec+len)-vec;
  for(int i=0; i<len; i++)
  	mp[vec[i]]=i+1;
  ```

+ A **Different** Segment Tree

  We make t[v] meaningful if and only if the subsegments are all the same (In this question means the posters are the same). So whenever we update to or query to any subsegment of vertex v, we set it's val t[v]=0 to let it meaningless. When we query to a segment which is meaningless, we must query to its subsequent rather than returning t[v] directly.

+ **My code gets RE!!!** But I don't know where I'm wrong...I had made 100 random test cases but I hadn't found my fault. **:(**

### RE Code

```c++
#include<iostream>
#include<cstdio>
#include<algorithm>
#include<string>
#include<cstring>

using namespace std;

const int maxn=1e4;
int t[maxn<<4];
int lazy[maxn<<2];
int mp[maxn<<2];
bool vis[maxn<<3];
int vec[maxn<<2];
int input[maxn<<1][2];

void push(int v){
	if(lazy[v]==0) return;
	lazy[v<<1]=lazy[v<<1|1]=lazy[v];
	t[v<<1]=t[v<<1|1]=lazy[v];
	lazy[v]=0;
	t[v]=0;
}
void build(int v, int tl, int tr){
	if(tl==tr) t[v]=0;
	else{
		int mid=tl+tr>>1;
		build(v<<1, tl, mid);
		build(v<<1|1, mid+1, tr);
	}
	t[v]=0;
}
void update(int v, int tl, int tr, int l, int r, int val){
	if(l>r) return;
	if(tl==l&&tr==r) t[v]=lazy[v]=val;
	else{
		push(v);
		int mid=tl+tr>>1;
		update(v<<1, tl, mid, l, min(r, mid), val);
		update(v<<1|1, mid+1, tr, max(l, mid+1), r, val);
		t[v]=0;
	}
}
int ans;
void query(int v, int tl, int tr, int l, int r){
	if(l>r) return;
	if(t[v]!=0){
		if(!vis[t[v]]){
			vis[t[v]]=true;
			ans++;
		}
		return;
	}
	if(tl==tr) return;
	int mid=tl+tr>>1;
	query(v<<1, tl, mid, l, min(r, mid));
	query(v<<1|1, mid+1, tr, max(l, mid+1), r);
}
int main(){
	int times;
	scanf("%d", &times);
	while(times--){
		ans=0;
		memset(vis, false, sizeof(vis));
		int len;
		scanf("%d", &len);
		int size=len;
		for(int i=0; i<len; i++){
			int a, b;
			scanf("%d%d", &a, &b);
			input[i][0]=vec[i<<1]=a;
			input[i][1]=vec[i<<1|1]=b;
		}
        /*
        	Discretization
        */
		sort(vec, vec+(len<<1));
		len=unique(vec, vec+(len<<1))-vec;
		for(int i=len-2; i>=0; i--){
			if(vec[i+1]-vec[i]>1) {
				vec[len++]=vec[i+1]+vec[i]>>1;
			}
		}
		sort(vec, vec+len);
		len=unique(vec, vec+len)-vec;
		for(int i=0; i<len; i++)
			mp[vec[i]]=i+1;
		build(1, 1, len);
		for(int i=0; i<size; i++)
			update(1, 1, len, mp[input[i][0]], mp[input[i][1]], i+1);
		query(1, 1, len, 1, len);
		printf("%d\n", ans);
	}
	return 0;
}

```

