# Segment Tree

**Attention that it's a simplified version of content on [web](https://cp-algorithms.com/data_structures/segment_tree.html#toc-tgt-9) . For more details can go to this [web](https://cp-algorithms.com/data_structures/segment_tree.html#toc-tgt-9) .** 

[TOC]



## Simplest form of a Segment Tree

### Structure of the Segment Tree

A Segment Tree is a data structure that allows answering range queries over an array effectively, while still being flexible enough to allow modifying the array. 

We compute and store the sum of the elements of the whole array( the sum of the segment $a[0...n-1]$ ). We then split the array into two halves $a[0...n/2]$ and $a[n/2...n-1]$ and compute the sum of each halve and store them.

Consider the number of nodes in the worst case:

![](./sum-segment-tree.png)

The number of nodes in the worst case can be estimated by the sum $\sum_{i=0}^{k=\left \lceil \log n \right \rceil}{2^k}=2^{\left \lceil \log n \right \rceil +1} <4n$   

### Construction

So, we store the Segment Tree simply as an array t[] with a size of four times the input size n:

```c++
int n, t[4*MAXN]
```

Construction:

```c++
/*
	a[]:the input array
	v:the current index
	left, right:the left boundary of the segment
*/
void build(int a[], int v, int left, int right){
	if(left==right){
		t[v]=a[right];
	}else{
		int mid=(left+right)/2;
		build(a, v*2, left, mid);
		build(a, v*2+1, mid+1, right);
		t[v]=t[v*2]+t[v*2+1];
	}
}
```

### Sum queries

```c++
/*
	l, r:the boundary of the query
	v:the current index
	left, right:the boundaries of the segment
*/
int sum(int v, int left, int right, int l, int r){
	if(l>r) return 0;
	if(l==left&&r==right){
		return t[v];
	}
	int mid=(left+right)/2;
	return sum(v*2, left, mid, l, min(r, mid))
			+sum(v*2+1, mid+1, right, max(l, mid+1), r);
}
```

### Update queries

```c++
/*
	v:the current index
	left, right:the boundaries of the segment
	pos:the index to be updated
	new_val:updated value
*/
void update(int v, int left, int right, int pos, int new_val){
	if(left==right) t[v]=new_val;
	else{
		int mid=(left+right)/2;
		if(pos<=mid) update(v*2, left, mid, pos, new_val);
		else update(v*2+1, mid+1, right, pos, new_val);
		t[v]=t[v*2]+t[v*2+1];
	}
}
```

## Advanced versions of Segment Trees

### More complex queries

#### Finding the maximum and the number of times it appears

This task is very similar to the previous one.  Just need to the way t[v] is computed in the **build** and **update** functions. 

Consider another question,  how to find the maximum and the the numbers  of its occurrences?

```c++
pair<int, int> t[4*MAXN];
pair<int, int> combine(pair<int, int> a, pair<int, int> b){
	if(a.first>b.first) return a;
	if(b.first>a.first) return b;
	return make_pair(a.first, a.second+b.second);
}
void build(int a[], int v, int left, int right){
	if(left==right) t[v]=make_pair(a[left], 1);
	else{
		int mid=(left+right)/2;
		build(a, v*2, left, mid);
		build(a, v*2+1, mid+1, right);
		t[v]=combine(t[v*2], t[v*2+1]);
	}
}
pair<int, int> get_max(int v, int left, int right, int l, int r){
	if(l>r) retrun make_pair(INT_MIN, 0);
	if(l==left&&r==right) return t[v];
	int mid=(left+right)/2;
	return combine(get_max(v*2, left, mid, l, mid(r, mid),
					get_max(v*2+1, mid+1, right, mid(l, mid+1), r));
}
void update(int v, int left, int right, int pos, int new_val){
	if(left==right) t[v]=make_pair(new_val, 1);
	else{
		int mid=(left+right)/2;
		if(pos<=mid) update(v, left, mid, pos, new_val);
		else update(v, mid+1, right, pos, new_val);
		t[v]=combine(t[v*2], y[v*2+1]);
	}
}
```

#### Compute the greatest common divisor / least common multiple

Notice that :
$$
GCD(x_1, x_2, x_3, ..., x_n)=GCD(GCD(x_1, x_2), GCD(x_3, x_4),...)
$$
We can reduce computation, using:
$$
GCD(x_1, x_2, x_3, ..., x_n)=GCD(x_1, x_2-x_1, x_3-x_2, ...)
$$
And we can compute *GCD* by **更相减损法**:

```c++
int gcd(int a, int b){
	return b?gcd(b, a%b):a;
}
```

For *LCM*:

```c++
int lcm(int a, int b){
	return a*b/gcd(a,b);
}
```

#### Counting the number of zeros, searching for the k-th zero

t[i] ( not a leaf ) stores the number of zeros in each segment.

```c++
int find_kth(int v, int left, int right, int k){
	if(k>t[v]) return -1;
	if(left==right) return left;
	int mid=(left+right)/2;
	if(t[v*2]>=k) return find_kth(v*2, left, mid, k);
	else return find_kth(v*2+1, mid+1, right, k-t[v*2]);
}
```

#### Searching for an array prefix with a given amount

By moving each time to the left or the right,  depending on the left child.

#### Finding subsegments with the maximal sum

$a[l...r]$ is given for each queries, we have to find a subsegment $a[l'...r']$ of which  the sum is maximal.

This time we will store four values for each vertex: the sum of the segment, the maximum prefix sum, the maximum suffix sum, and the sum of the maximal subsegment in it.

```c++
struct date{
    int sum, pref, suff, ans;
};
data combine(data l, data r){
    data res;
    res.sum=l.sum+r.sum;
    res.pref=max(l.pref, l.sum+r.pref);
    res.suff=max(r.suff, r.sum+l.suff);
    res.ans=max(max(l.ans, r.ans), l.suff+r.pref);
    return res;
}
data make_data(int val){
    data res;
    res.sum=val;
    res.pref=res.suff=res.ans=max(0, val);
    return res;
}
void build(int a[], int v, int tl, int tr){
    if(tl==tr) t[v]=make_data(a[tl]);
    else{
        int mid=(tl+tr)/2;
        build(a, v*2, tl, mid);
        build(a, v*2+1, mid+1, tr);
        t[v]=combine(t[v*2], t[v*2+1]);
    }
}
void update(int v, int tl, int tr, int pos, int new_val){
    if(tl==tr) t[v]=make_data(new_val);
    else{
        int mid=(tl+tr)/2;
        if(pos<=mid) update(v*2, tl, mid, pos, new_val);
        else update(v*2+1, mid+1, tr, pos, new_val);
        t[v]=combine(t[v*2], t[v*2+1]);
    }
}
data query(int v, int tl, int tr, int l, int r){
    if(l>r) return make_data(0);
    if(l==tl&&r==tr) return t[v];
    int mid=(tl+tr)/2;
    return combine(query(v*2, tl, mid, l, min(r, mid)),
                  query(v*2+1, mid+1, tr, max(l, mid+1), r);
}
```

### Saving the entire subarrays in each vertex

This time, each vertex of the segment tree don't store the compressed information but all element of the segment. Thus the root of the Segment Tree will store all elements of the array, the left child vertex will store the first half of the array, the right vertex the second half, and so on.

This kind of Segment Tree costs $O(nlog n)$ memory. So it cost only slightly more memory than the usual Segment Tree.

#### Find the smallest number greater or equal to a specified number. No modification queries.

For given numbers $(l , r, x)$ we have to find the minimal number in the segment $a[l...r]$ which is greater than or equal to $x$ .

```c++
vector<int> t[4*MAXN];

void build(int a[], int v, int tl, int tr){
    if(tl==tr) t[v]=vector<int>(1, a[tl]);
    else{
        int mid=(tl+tr)/2;
        build(a, v*2, tl, mid);
        build(a, v*2+1, mid+1, tr);
        merge(t[v*2].begin(), t[v*2].end(), t[v*2+1].begin(), t[v*2+1].end(), back_inserter(t[v]));
    }
}

int query(int v, int tl, int tr, int l, int r, int x){
    if(l>r) return INT_MAX;
    if(l==tl&&r==tr){
        vector<int>::iterator pos=lower_bound(t[v].begin(), t[v].end(), x);
        if(pos!=t[v].end()) return *pos;
        else return INT_MAX;
    }
    int mid=(tl+tr)/2;
    return min(query(v*2, tl, mid, l, min(r, mid), x), 
              query(v*2+1, mid+1, tr, max(l, mid+1), r, x));
}

```

Time Complexity: $O(log^2n)$ 

#### Find the smallest number greater or equal to a specified number. With modification queries.

With modification, it's hard to modify the array between answering queries.

In order to deal with it, we can store the array with **multiset** but not **vector**.

It's easy to quickly search for numbers, delete numbers, and insert new numbers. But it increase the time complexity when finding the lower_bound of a segment( you need to sort it first ) .This leads to a construction time of  $O(nlog^2n)$  .

```c++
void update(int v, int tl, int tr, int pos, int new_val){
    t[v].erase(t[v].find(a[pos]));
    t[v].insert(new_val);
    if(tl!=tr){
        int mid=(tl+tr)/2;
        if(pos<=mid) update(v*2, tl, mid, pos, new_val);
        else update(v*2+1, mid+1, tr, pos, new_val);
    }
    else a[pos]=new_val;
}
```

Processing of this modification query also takes $O(log^2n)$ time.

Topic for acceleration: **fractional cascading**

### Range updates (Lazy Propagation)

Modification queries discussed above only effected a single element of the array each. And for modification queries to an entire segment of contiguous elements, we can use **Lazy Propagation**.

#### Addition on segments

For given numbers $(l, r, x)$ , the modification queries should add a number $x$ to all numbers in the segment $a[l...r]$ . The second queries are supposed to answer for the value of $a[i]$.

To make the addition query efficient, we store at each vertex in the Segment Tree how many we should add to all numbers in the segment. We call it **Lazy Flag**. In usual, we don't need to modify one's **Lazy Flag** until we need query for the subsegment of that. When it come to a second query, we just pass down the **Lazy Flag** until meeting the queried segment. Thus we don't have to change all $O(n)$ values, but only $O(log n)$ values.

```c++
/*
	in this block, t[i] (not a leaf) is used as Lazy Flag
*/
void build(int a[], int v, int tl, int tr){
    if(tl==tr) t[v]=a[tl];
    else{
        int mid=(tl+tr)/2;
        build(a, v*2, tl, mid);
        build(a, v*2+1, mid+1, tr);
        t[v]=0;
    }
}
void update(int v, int tl, int tr, int l, int r, int add){
    if(l>r) return;
    if(l==tl&&r==tr) t[v]+=add;
    else{
     	int mid=(tl+tr)/2;
        update(v*2, tl, mid, l, min(r, mid), add);
        update(v*2+1, mid+1, tr, max(l, mid+1), r, add);
    }
}
int get(int v, int tl, int tr, int pos){
    if(tl==tr) return t[v];
    int mid=(tl+tr)/2;
    if(pos<=mid) return t[v]+get(v*2, tl, mid, pos);
    else return t[v]+get(v*2+1, mid+1, tr, pos);
}
```

#### Adding on segments, querying for maximum

We keep store an additional value (**lazy**) for each vertex. In this value we store the addends we haven't propagated to the child vertices. Before traversing to a child vertex, we call **push** and propagate the value to both children.

```c++
void push(int v){
    t[v*2]+=lazy[v];
    lazy[v*2]+=lazy[v];
    t[v*2+1]+=lazy[v];
    lazy[v*2+1]+=lazy[v];
    lazy[v]=0;
}
void update(int v, int tl, int tr, int l, int r, int addend){
    if(l>r) return;
    if(l==tl&&r==tr){
        lazy[v]+=addend;
        t[v]+=addend;
    }else{
        push(v);
        int mid=(tl+tr)/2;
        update(v*2, tl, mid, l, min(r, mid), addend);
        update(v*2+1, mid+1, tr, max(l, mid+1), r, addend);
        t[v]=max(t[v*2], t[v*2+1]);
    }
}
int query(int v, int tl, int tr, int l, int r){
    if(l>r) return INT_MIN;
    if(l==tl&&r==tr) return t[v];
    push(v);
    int mid=(tl+tr)/2;
    return max(query(v, tl, mid, l, min(r, mid)),
              query(v, mid+1, tr, max(l, mid+1), r));
}
```

### Generalization to higher dimensions

#### Simple 2D Segment Tree

A matrix **$a[0...n-1, 0...m-1]$** is given, and we have to find the sum (or minimum/maximum) on some submatrix  $a[x_1...x_2, y_1...y_2] $ , as well as perform modifications of individual matrix elements.

So we build a 2D Segment Tree using the first coordinate $(x)$ , then the second $(y)$ .

Attention that, we should make sure that doing assignment for $range\,y$  first and then for $range\,x$.

```c++
void build_y(int vx, int lx, int rx, int vy, int ly, int ry){
    if(ly==ry){
        if(lx==rx) t[vx][vy]=a[lx][ly];
        else t[vx][vy]=t[vx*2][vy]+t[vx*2+1][vy];
    }else{
        int my=(ly+ry)/2;
        build_y(vx, lx, rx, vy*2, ly, my);
        build_y(vx, lx, rx, vy*2+1, my+1, ry);
        t[vx][vy]=t[vx][vy*2]+t[vx][vy*2+1];
    }
}
void build_x(int vx, int lx, int rx){
    if(lx!=rx){
        int mx=(lx+rx)/2;
        build_x(vx*2, lx, mx);
        build_x(vx*2+1, mx+1, rx);
    }
    build_y(vx, lx, rx, 1, 0, m-1);
}
int sum_y(int vx, int vy, int tly, int try_, int ly, int ry){
    if(ly>ry) return 0;
    if(ly==tly&&ry==try_) return t[vx][vy];
    int mid=(tly+try_)/2;
    return sum_y(vx, vy*2, tly, mid, ly, min(ry, mid))
        +sum_y(vx, vy*2+1, mid+1, try_, max(ly, mid+1), ry);
}
/*
	Time Complexity: O(log n*log m)	
*/
int sum_x(int vx, int tlx, int trx, int lx, int rx, int ly, int ry){
    if(lx>rx) return 0;
    if(tlx==lx&&trx==rx) return sum_y(vx, 1, 0, m-1, ly, ry);
    else{
        int mid=(tlx+trx)/2;
        return sum_x(vx*2, tlx, mid, lx, min(rx, mid), ly, ry)
            +sum_x(vx*2+1, mid+1, trx, max(lx, mid+1), rx, ly, ry);
    }
}
/*
	Update function is similar with Build function
*/
void update_y(int vx, int lx, int rx, int vy, int ly, int ry, int x, int y, int new_val){
    if(ly==ry){
        if(lx==rx) t[vx][vy]=new_val;
        else t[vx][vy]=t[vx*2][vy]+t[vx*2+1][vy];
    }else{
        int mid=(ly+ry)/2;
        if(y<=mid) update_y(vx, vy*2, lx, rx, ly, mid, x, y, new_val);
        else update_y(vx, vy*2+1, lx, rx, mid+1, ry, x, y, new_val);
        t[vx][vy]=t[vx][vy*2]+t[vx][vy*2+1];
    }
}
void update_x(int vx, int lx, int rx, int x, int y, int new_val){
    if(lx!=rx){
        int mid=(lx+rx)/2;
        if(x<=mid) update_x(vx*2, lx, mid, x, y, new_val);
        else update_x(vx*2+1, mid+1, rx, x, y, new_val);
    }
    update_y(vx, lx, rx, 1, 0, m-1, x, y, new_val);
}
```

















