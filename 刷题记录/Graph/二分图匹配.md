# 二分图匹配

```java
boolean dfs(int u){
    int v;
    for(v=0; v<vN; v++){
        if(g[u][v]&&!used[v]){
            used[v]=true;
            if(linker[v]==-1||dfs(linker[v])){
                linker[v]=u;
                return true;
            }
        }
    }
    return false;
}
int hungary(){
    int res=0;
    int u;
    Arrays.fill(linker, -1);
    for(u=0; u<uN; u++){
        Arrays.fill(used, 0);
        if(dfs(u)) res++;
    }
    return res;
}
```

