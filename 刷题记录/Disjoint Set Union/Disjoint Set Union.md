# Disjoint Set Union

## 基本实现

```java
void make_set(int v){
    parent[v]=v;
}
int find_set(int v){
    if(v==parent[v]) return v;
    return find_set(parent[v]);
}
void union(int a, int b){
    a=find_set(a);
    b=find_set(b);
    if(a!=b) parent[a]=b;
}
```

## 路径压缩优化

```java
int find_set(int v){
    if(v==parent[v]) return v;
    int root=v;
    while(root!=parent[root]) root=parent[root];
    while(v!=root){
        int tmp=parent[v];
        parent[v]=root;
        v=tmp;
    }
    return root;
}
```

## 根据size或rank合并

```java
void make_set(int v){
    parent[v]=v;
    size[v]=1;
}
void union(int a, int b){
    a=find_set(a);
    b=find_set(b);
    if(a!=b){
        if(size[a]<size[b]) swap(a, b);
        parent[b]=a;
        size[a]+=size[b];
    }
}
```

```java
void make_set(int v){
    parent[v]=v;
    rank[v]=0;
}
void union(int a, int b){
    a=find_set(a);
    b=find_set(b);
    if(a!=b){
        if(rank[a]<rank[b]) swap(a, b);
        parent[b]=a;
        if(rank[a]==rank[b]) rank[a]++;
    }
}

```

## 时间复杂度

实现了路径压缩以及优化合并后的平均耗费$O(1)$。

## 经验

凡是等价的问题都可以用DSU尝试。

要定义出union操作是什么，并且union的操作不能遗漏。例如用DSU求数组元素中能够组成最长连续的序列长度，那么所有潜在的可以union的操作都需要尝试，这些union操作一般统一在一起完成，例如遍历数组，对每个元素尝试进行相邻union操作。