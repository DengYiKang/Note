# Lexicographical Numbers

### 问题

求`1～n`的字典序排列。

### 解决方案：dfs，$O(n)$

```java
class Solution {
    List<Integer> ans;
    public List<Integer> lexicalOrder(int n) {
        ans=new ArrayList<>();
        StringBuilder sb=new StringBuilder();
        for(int i=1; i<=9&&i<=n; i++){
            sb.append(String.valueOf(i));
            dfs(sb, n);
            sb.deleteCharAt(sb.length()-1);
        }
        return ans;
    }
    void dfs(StringBuilder sb, int n){
        int val=Integer.valueOf(sb.toString());
        if(val>n) return;
        ans.add(val);
        for(int i=0; i<=9; i++){
            sb.append(String.valueOf(i));
            dfs(sb, n);
            sb.deleteCharAt(sb.length()-1);
        }
    }
}
```

```java
//之前太蠢了，用StringBuilder来表示当前数，可以直接用int的
class Solution {
    List<Integer> ans;
    public List<Integer> lexicalOrder(int n) {
        ans=new ArrayList<>();
        StringBuilder sb=new StringBuilder();
        for(int i=1; i<=9&&i<=n; i++){
            dfs(i, n);
        }
        return ans;
    }
    void dfs(int val, int n){
        if(val>n) return;
        ans.add(val);
        for(int i=0; i<=9; i++){
            dfs(val*10+i, n);
        }
    }
}
```

