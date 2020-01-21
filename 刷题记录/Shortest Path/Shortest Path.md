# Shortest Path

## floyd

```java
void floyd(){
    for(int k=0; k<n; k++)
        for(int i=0; i<n; i++)
            for(int j=0; j<n; j++)
                dist[i][j]=min(dist[i][j], dist[i][k]+dist[k][j]);
}
```

## dijkstra

```java
static final int INF=0x3f3f3f3f;
boolean[] vis, dist;
int[][] mp;
int n;
int[] w, num, weight;
/*there exists at least two shortest paths between start and end*/
/*
w[i]:the maximum of weight on the path
num[i]:the amount of the shortest paths to i
*/
void init(){
    //init whith space...
    Arrays.fill(mp, INF);
    Arrays.fill(dist, INF);
    Arrays.fill(vis, false);
}
void dijkstra(int s){
    dist[s]=0;
    w[s]=weight[maxn];
    num[s]=1;
    for(int i=0; i<n; i++){
        int Min=INF, u=-1;
        for(int j=0; j<n; j++){
            if(!vis[j]&&dist[j]<Min){
                Min=dist[j];
                u=j;
            }
        }
        if(u==-1) return;
        vis[u]=true;
        for(int j=0; j<n; j++){
            if(vis[j]||dist[u][j]==INF) continue;
            if(dist[u]+mp[u][j]<dist[j]){
                dist[j]=dist[u]+mp[u][j];
                num[j]=num[u];
                w[j]=w[u]+weight[j];
            }
            else if(dist[u]+mp[u][j]==dist[j]){
                num[j]+=num[u];
                w[j]=max(w[j], w[u]+weight[j]);
            }
        }
    }
}
```



