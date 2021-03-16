# Trapping Rain Water

### 问题

`int[] height`表示间距为1的直方图的高。问这些直方图能容下多少水。

![img](../../pic/66.png)

### 解决方案：stack，时间复杂度$O(n)$，空间复杂度$O(n)$

当前的高度为h[i]，往前搜索，如果有比自己高的数h[j]，那么ans+=(h[i]-land)*(i-j-1)。这个land是陆地的高度；如果比自己小的数h[k]，那么ans+=(h[k]-land)*(i-k-1)，同时更新land为h[k]，pop h[k]。最后更新land为当前高度。

![](/home/yikang/Picture/2021-03-16 09-52-04屏幕截图.png)

对于这种情况，land为1，因为在这之前下面的倒T的凹槽部分的体积已经被计算进去了，可以将它看为已经填满的陆地，因此需要减去land这个值。

```java
class Solution {
        public int trap(int[] height) {
                Stack<int[]> stack=new Stack<>();
                int ans=0, land=0;
                for(int i=0; i<height.length; i++){
                        if(height[i]==0){
                                land=0;
                                continue;
                        }
                        while(!stack.isEmpty()&&stack.peek()[0]<=height[i]){
                                int[] left=stack.pop();
                                ans+=(left[0]-land)*(i-left[1]-1);
                                land=left[0];
                        }
                        if(!stack.isEmpty()&&stack.peek()[0]>height[i]){
                                ans+=(height[i]-land)*(i-stack.peek()[1]-1);
                        }
                        stack.push(new int[]{height[i], i});
                        land=height[i];
                }
                return ans;
        }
}
```

