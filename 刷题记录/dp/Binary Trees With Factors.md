# Binary Trees With Factors

### 问题

一个整型数组`arr`，元素不重复，且都大于1。可以重复使用元素组成二叉树，要求父节点是子节点的乘积。求问有多少种构造方法，结果取模10e9+7。

### 解决方案：dp， 时间复杂度$O(n^2)$

现将`arr`排序，`dp[i]`表示以`arr[i]`为root的构造方法种数，那么

`dp[i]=sum(dp[j]*dp[k],if i==j*k and j==k)+sum(dp[j]*dp[k]*2, if i==j*k and j!=k) `

```java
class Solution {
    public int numFactoredBinaryTrees(int[] arr) {
        long module=(long) Math.pow(10, 9)+7;
        long[] dp=new long[arr.length];
        Arrays.fill(dp, 1);
        HashMap<Integer, Integer> mp=new HashMap<>();
        Arrays.sort(arr);
        mp.put(arr[0], 0);
        for(int i=1; i<arr.length; i++){
            boolean[] vis=new boolean[i];
            for(int j=0; j<i; j++){
                if(vis[j]) continue;
                if(arr[i]%arr[j]!=0) continue;
                int quo=arr[i]/arr[j];
                if(!mp.containsKey(quo)) continue;
                if(quo==arr[j]){
                    dp[i]+=(dp[mp.get(quo)]*dp[j])%module;
                }else{
                    dp[i]+=(dp[mp.get(quo)]*dp[j]*2)%module;
                }
                dp[i]%=module;
                vis[j]=vis[mp.get(quo)]=true;
            }
            mp.put(arr[i], i);
        }
        long res=0;
        for(int i=0; i<dp.length; i++){
            res+=dp[i]%module;
            res=res%module;
        }
        return (int) res;
    }
}
```

