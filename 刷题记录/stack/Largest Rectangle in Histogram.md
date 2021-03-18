# Largest Rectangle in Histogram

### 问题

给一个直方图，求在直方图里面积最大的长方形（包含正方形）。

![img](../../pic/67.jpg)

### 解决方案：stack，时间复杂度$O(n^2)$

维护一个栈，保存着之前存放的块。对于当前`heights[j]`，向前搜索，如果搜索到的`heights[i]`比`heights[j]`更大，那么对于后续搜索的`heights[k]`，比`heights[i]`更小的`heights[j]`将会成为短板，因此在后续比较中`heights[i]`没有用了，因此需要将`heights[i]`出栈。

因此，栈中的块的高度是严格递增的。

但注意，虽然将栈中的所有高度大于自身的块出栈，但还是要从头比较的。

因为需要遍历，因此使用ArrayList来模拟栈，同时提供遍历的功能。

计算面积的时候得注意，弹出栈的过程和从头遍历的过程的宽度的计算公式是不一样的。

```java
class Solution {
        public int largestRectangleArea(int[] heights) {
                List<int[]> list=new ArrayList<>();
                int ans=0;
                for(int i=0; i<heights.length; i++){
                        if(list.size()!=0){
                                int tail=list.size()-1;
                                while(tail>=0&&list.get(tail)[0]>=heights[i]) list.remove(tail--);
                                int area=0;
                            	//弹出栈过程中的面积计算公式
                                if(tail>=0) area=((i-list.get(tail)[1])*heights[i]);
                                else area=(i+1)*heights[i];
                                for(int k=0; k<list.size(); k++){
                                    //遍历过程中的面积计算公式
                                        if(k==0) area=Math.max(area, list.get(k)[0]*(i+1));
                                        else area=Math.max(area, list.get(k)[0]*(i-list.get(k-1)[1]));
                                }
                                ans=Math.max(area, ans);
                        }else{
                                ans=Math.max(heights[i]*(i+1), ans);
                        }
                        list.add(new int[]{heights[i], i});
                }
                return ans;
        }
}
```

