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

从上面的分析中，可以看出，可以利用两个不等关系来判断当前pos是左侧的false还是右侧的false还是中间的true。

因为返回的是最大值h=len-pos，因此需要找到连续true的最左侧的true，使得pos最小。

那么只需要确定评判函数，使得能正确识别好出中间的true和右侧的false，直接套用连续的第一个二分查找模板即可：

```java
class Solution {
    public int hIndex(int[] c) {
        int l=0, r=c.length-1;
        while(l<=r){
            int mid=(l+r)>>1;
            if(eq1(c, mid)){
                r=mid-1;
            }else{
                l=mid+1;
            }
        }
        return c.length-l;
    }
    boolean eq1(int[] c, int pos){
        return c[pos]>=c.length-pos;
    }
    //这个不等式没有用到
    boolean eq2(int[] c, int pos){
        if(pos==0) return true;
        return c[pos-1]<=c.length-pos;
    }
    
}
```

