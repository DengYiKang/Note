# Longest Mountain in Array

### 问题

在整型数组`arr`中找到一个子数组`A`使得：

+ `A.length>=3`
+ 存在`i`，使得`A[0]<A[1]<...<A[i]>A[i+1]>...>A[A.length-1]`

求`A`的最长长度。

### 解决方案：滑动窗口，时间复杂度$O(n)$

问题在于如何判断窗口内的子数组是哪种状态，是处于递增状态还是递减状态（滑动窗口维护的都是合法的子数组）。

可以设立两个标志，`flag1=true`表示已经递增过了，`flag2=true`表示已经递减过了。

```java
class Solution {
    public int longestMountain(int[] arr) {
        int lo=0, ans=0;
        boolean flag1=false, flag2=false;
        for(int i=1; i<arr.length; i++){
            if(arr[i-1]<arr[i]){
                if(flag1&&flag2){
                    ans=Math.max(ans, i-lo);
                    lo=i-1;
                    flag1=true;
                    flag2=false;
                }else if(!flag1&&flag2){
                    lo=i-1;
                    flag1=true;
                    flag2=false;
                }else if(!flag1&&!flag2){
                    flag1=true;
                }
            }
            else if(arr[i-1]>arr[i]){
                if(!flag1&&flag2){
                    lo=i;
                    flag1=flag2=false;
                }else if(flag1&&!flag2){
                    flag2=true;
                }else if(!flag1&&!flag2){
                    lo=i;
                }
            }else if(arr[i-1]==arr[i]){
                if(flag1&&flag2){
                    ans=Math.max(ans, i-lo);
                }
                lo=i;
                flag1=flag2=false;
            }
        }
        if(flag1&&flag2) ans=Math.max(ans, arr.length-lo);
        return ans;
    }
}
```

