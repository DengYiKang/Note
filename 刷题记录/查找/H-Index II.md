# H-Index II

### 问题

输入一个升序得到数组c，求出一个值h，使得数组c中h个元素大于等于h，其余的元素小于等于h。若有多个h，求最大的那个。

### 解决方案：折半查找，时间复杂度$O(n)$，空间复杂度$O(1)$ 

设一个索引为`mid`，那么`c[mid]~c[len-1]`的长度为`len-mid`，在这里把`len-mid`视为`h`。对`mid`进行搜索。

首先判断该问题是否符合折半查找的特征：

若`len-mid`为符合题意的`h`，那么有`c[mid]>=len-mid`和`mid-1>=0&&c[mid-1]<=len-mid`。

考虑不符的情况：

+ `c[mid-1]<=c[mid]<len-mid`:此时`mid`显然只能增加
+ `c[mid]>=len-mid but c[mid-1]>len-mid`：此时`mid`显然只能减少

因此该问题适用折半查找。

```java
class Solution {
    public int hIndex(int[] c) {
        int lo=0, hi=c.length-1, len=c.length, ans=0;
        while(lo<=hi){
            int mid=(lo+hi)/2;
            if(c[mid]<len-mid) lo=mid+1;
            else if(mid-1>=0&&c[mid]>=len-mid&&c[mid-1]>len-mid) hi=mid-1;
            else {
                ans=Math.max(ans, len-mid);
                //符合题意,继续搜索更大的h值
                hi--;
            }
        }
        return ans;
    }
}
```

