# Rectangle Area

### 问题

输入四个点坐标，对应两个矩形，求两个矩形围成的面积

![Rectangle Area](../pic/2.png)

### 解决方案

```java
class Solution {
    public int computeArea(int A, int B, int C, int D, int E, int F, int G, int H) {
        int areA=(C-A)*(D-B);
        int areB=(G-E)*(H-F);
        int left=Math.max(A, E);
        int right=Math.min(C, G);
        int top=Math.min(D, H);
        int bottom=Math.max(B, F);
        int overlap=0;
        if(left<right&&top>bottom) overlap=(right-left)*(top-bottom);
        return areA+areB-overlap;
    }
}
```



