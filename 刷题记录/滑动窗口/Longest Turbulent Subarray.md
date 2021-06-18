# Longest Turbulent Subarray

### 问题

找到一个数组的最长的turbulent的连续子数组，turbulent指相邻的三个点，前两点的连线的梯度与后两点的正负不同，梯度不能为0。

### 解决方案：滑动窗口，时间复杂度$O(n)$

```java
class Solution {
    public int maxTurbulenceSize(int[] arr) {
        int status=-1;
        int l=0, r=1, ans=1;
        while(r<arr.length){
            if(status==-1){
                if(arr[r]==arr[r-1]) l=r;
                else status=arr[r]>arr[r-1]?r%2:1-(r%2);
            }else{
                if(status==r%2&&arr[r]<=arr[r-1]
                  || status!=r%2&&arr[r]>=arr[r-1]){
                    ans=Math.max(ans, r-l);
                    if(arr[r-1]==arr[r]){
                        l=r;
                        status=-1;
                    }else{
                        l=r-1;
                        status=arr[r]>arr[r-1]?r%2:1-(r%2);    
                    }
                }
            }    
            r++;
        }
        return Math.max(ans, r-l);
    }
}
```



