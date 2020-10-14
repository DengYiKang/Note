# Rotate Function

### 问题

现有一个整型数组$A$，$n$为其长度。$B_k$是$A$在第$k$位上旋转得到，定义$F(k)$：
$$
F(k)=0*B_k[0]+1*B_k[1]+...+(n-1)*B_k[n-1]
$$
求$max\{F(k)\}$。

**Example:**

```
A = [4, 3, 2, 6]

F(0) = (0 * 4) + (1 * 3) + (2 * 2) + (3 * 6) = 0 + 3 + 4 + 18 = 25
F(1) = (0 * 6) + (1 * 4) + (2 * 3) + (3 * 2) = 0 + 4 + 6 + 6 = 16
F(2) = (0 * 2) + (1 * 6) + (2 * 4) + (3 * 3) = 0 + 6 + 8 + 9 = 23
F(3) = (0 * 3) + (1 * 2) + (2 * 6) + (3 * 4) = 0 + 2 + 12 + 12 = 26

So the maximum value of F(0), F(1), F(2), F(3) is F(3) = 26.
```

### 解决方案：时间复杂度$O(n)$

在k位翻转，那么$B_k$的首位应该为$A[(n-k)\%n]$。

注意到：
$$
F(i)-F(i-1)=\sum_{k=0}^{n-1}A[k]-nA[(n-i)\%n]
$$
那么令$diff=\Sigma (F(i)-F(i-1))$，遍历中取得$max\{diff\}$，在加上offset $F(0)$即可。

```java
class Solution {
    public int maxRotateFunction(int[] A) {
        int n=A.length;
        if(n==0) return 0;
        long diff=0, sum=0L;
        long max_diff=0;
        for(int i=0; i<n; i++){
            sum+=(long) A[i];
        }
        for(int i=1; i<n; i++){
            diff+=sum-n*A[(n-i)%n];
            max_diff=Math.max(diff, max_diff);
        }
        long ans=max_diff;
        for(int i=0; i<n; i++){
            ans+=i*A[i];
        }
        return (int) ans;
    }
}
```



