# 2 Keys Keyboard

### 问题

当前的屏幕上有一个`A`，你可以执行以下操作，每个操作为一步：

+ `copy all`：将屏幕上所有的字符串拷贝到剪切板
+ `paste`：将剪切板里的内容粘贴到屏幕上，append方式，剪切板内容还在

要求最终屏幕上有`n`个`A`，求最小步长。

### 解决方案：

#### 一、dfs

维护一个外部变量`min_val`，保存当前搜索的最小步长，作为剪枝条件。

```java
class Solution {
    int min_val=Integer.MAX_VALUE;
    public int minSteps(int n) {
        if(n==1) return 0;
        min_val=Integer.MAX_VALUE;
        dfs(1, 1, n, 1);
        return min_val;
    }
    public void dfs(int step, int status, int target, int depth){
        status+=step;
        depth++;
        if(depth>min_val) return;
        if(status>target) return;
        if(status==target) {
            min_val=Math.min(min_val, depth);
            return;
        }
        if(step%2==0&&status%2==0&&target%2!=0) return;
        //注意copy也占一个步骤
        dfs(status, status, target, depth+1);
        dfs(step, status, target, depth);
        return;
    }
}
```

#### 二、math，时间复杂度$O(\sqrt n)$

将copy和paste的步骤分别用C和P表示，那么步骤序列以`C`最为划分边界，例如：

`CPPCPPPPCP`=>`[CPP][CPPPP][CP]`

假设每个划分产生的长度`length[i]`，每个划分的长度为`g[i]`，那么有`length[i]=g[0]*g[1]*...*g[i]`。

那么产生`length[i]`长度的字符串，需要的步长为：`g[0]+g[1]+...+g[i]`。

对于`g=p*q`，当`p>2, q>2`时，有`(p-1)(q-1)>0`，即`p+q<pq+1 <==> p+q<=pq`。因此对于`g[i]`，总是对他进行因式分解。因此这个问题转化为质因数分解的问题。

```java
class Solution {
    public int minSteps(int n) {
        int ans = 0, d = 2;
        while (n > 1) {
            while (n % d == 0) {
                ans += d;
                n /= d;
            }
            d++;
        }
        return ans;
    }
}
```

