# Satisfiability of Equality Equations

### 问题

输入多个不等式，长度为4，形如`a==b`、`a!=c`的形式，变量都是小写的单字符。问所给的不等式是否会发生冲突。

### 解决方案：并查集，时间复杂度$O(n)$

一种方法是先将所有的`==`进行union操作，将所有的`==`操作完之后，对所有的`!=`进行检查；

另一种方法是按顺序操作，对union操作，查看是否两root有`!=`的关系；对`!=`的关系转换成它们的root的之间的`!=`，用mp存起来。

这里采用的第一种。

```java
class Solution {
        int[] parent;
        public boolean equationsPossible(String[] equations) {
                parent=new int[26];
                for(int i=0; i<26; i++){
                        makeSet(i);
                }
                List<int[]> notEq=new ArrayList<>();
                for(String eq:equations){
                        int u=eq.charAt(0)-'a';
                        int v=eq.charAt(3)-'a';
                        if(eq.charAt(1)=='=') union(u, v);
                        else notEq.add(new int[]{u, v});
                }
                for(int[] x:notEq){
                        if(findRoot(x[0])==findRoot(x[1]))
                                return false;
                }
                return true;
        }
        private void makeSet(int v){
                parent[v]=v;
        }
        private int findRoot(int v){
                int root=v;
                while(parent[root]!=root) 
                        root=parent[root];
                while(parent[v]!=root){
                        int tmp=parent[v];
                        parent[v]=root;
                        v=tmp;
                }
                return root;
        }
        private void union(int u, int v){
                int root_u=findRoot(u);
                int root_v=findRoot(v);
                if(root_u!=root_v) 
                        parent[root_u]=root_v;
        }
}
```



