# Longest Turbulent Subarray

### 问题

找到一个数组的最长的turbulent的连续子数组，turbulent指相邻的三个点，前两点的连线的梯度与后两点的正负不同，梯度不能为0。

### 解决方案：滑动窗口，时间复杂度$O(n)$

要注意梯度不能为0。

有两种思路，一种重新从第二个点开始搜索，需要维护一个变量表示是否为开始，因为当前梯度值的正负无法判断；第二种先搜索合法的第二个点，将当前梯度值初始化后从第三个点开始搜索。这里使用的是第二种。

```java
class Solution {
        public int maxTurbulenceSize(int[] arr) {
                int lo=0, ans=1;
                boolean state=false;
                while(lo+1<arr.length&&arr[lo]==arr[lo+1]) lo++;
                if(lo+1>=arr.length) return ans;
                else state=arr[lo+1]-arr[lo]>0?true:false;
                for(int i=lo+2; i<arr.length; i++){
                        boolean new_state=arr[i]-arr[i-1]>0?true:false;
                        if(arr[i]==arr[i-1]){
                                ans=Math.max(ans, i-lo);
                                while(i<arr.length&&arr[i]==arr[i-1]) i++;
                                lo=i-1;
                                //这里要注意边界
                                if(i<arr.length) state=arr[i]-arr[i-1]>0?true:false;
                                else return Math.max(ans, i-lo);
                        }else if(new_state==state){
                                ans=Math.max(ans, i-lo);
                                lo=i-1;
                                state=new_state;
                        }else{
                                state=new_state;
                        }
                }
                ans=Math.max(ans, arr.length-lo);
                return ans;
        }
}
```

