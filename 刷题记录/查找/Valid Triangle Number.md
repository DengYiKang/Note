# Valid Triangle Number

### 问题

有一个数组`A`，求`A`中所有的能构成三角形的三元组。

### 解决方案：二分查找，时间复杂度$O(n^2logn)$

```java
public class Solution {
    int binarySearch(int nums[], int l, int r, int x) {
        while (r >= l && r < nums.length) {
            int mid = (l + r) / 2;
            if (nums[mid] >= x)
                r = mid - 1;
            else
                l = mid + 1;
        }
        return l;
    }
    public int triangleNumber(int[] nums) {
        int count = 0;
        Arrays.sort(nums);
        for (int i = 0; i < nums.length - 2; i++) {
            int k = i + 2;
            for (int j = i + 1; j < nums.length - 1 && nums[i] != 0; j++) {
                k = binarySearch(nums, k, nums.length - 1, nums[i] + nums[j]);
                count += k - j - 1;
            }
        }
        return count;
    }
}
```

另一种：

```java
class Solution {
    public int triangleNumber(int[] nums) {
        Arrays.sort(nums);
        int ans=0;
        for(int i=0; i<nums.length; i++){
            for(int j=i+1; j<nums.length; j++){
                ans+=search(nums, j+1, nums.length-1, nums[i]+nums[j]);
            }
        }
        return ans;
    }
    //x=a+b，x是不能取的，这里找的是连续x的第一个x的位置
    //如果不存在x，那么l就是比x大的第一个位置
    //因此<x的个数在两种情况下都是l-origin_l
    int search(int[] a, int l, int r, int x){
        int origin_l=l, origin_r=r;
        while(l<=r){
            int mid=(l+r)>>1;
            if(a[mid]>=x){
                r=mid-1;
            }else{
                l=mid+1;
            }
        }
        return l-origin_l;
    }
}
```

