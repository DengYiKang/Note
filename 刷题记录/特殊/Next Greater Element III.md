# Next Greater Element III

### 问题

给一个数`n`，对这个数进行重组，重组后的数`m`拥有的元素与`n`相同，`m>n`，求`m`的最小值（若`m>2^31-1`或无此数时输出-1）

### 解决方案：时间复杂度$O(n)$ 

从后往前遍历`n`，找到第一个位置`index`，使得这个位置的值小于上个位置的值，那么在`index`后面的位置中找到比它大的最小的数，替换它，然后对`index`后面的位置的数从小到大排序即可。建议用数组存，Arrays.sort方法支持部分排序。

```java
class Solution {
    public int nextGreaterElement(int n) {
        int tmp=n, len=0;
        while(tmp>0){
            tmp/=10;
            len++;
        }
        char[] nums=new char[len];
        for(int i=len-1; i>=0; i--){
            nums[i]=(char)(n%10+'0');
            n/=10;
        }
        int index=len-1;
        while(index>=0){
            if(index!=len-1&&nums[index]<nums[index+1]) break;
            index--;
        }
        if(index==-1) return -1;
        int replacedIndex=findMinIndex(index, nums);
        char t=nums[index];
        nums[index]=nums[replacedIndex];
        nums[replacedIndex]=t;
        Arrays.sort(nums, index+1, nums.length);
        long ans=0;
        for(char ch:nums){
            ans*=10;
            ans+=ch-'0';
        }
        return ans<=Integer.MAX_VALUE?(int)ans:-1;
    }
    public int findMinIndex(int offset, char[] nums){
        int minIndex=-1;
        char minVal=255;
        for(int i=offset+1; i<nums.length; i++){
            if(nums[i]>nums[offset]&&nums[i]<minVal){
                minIndex=i;
                minVal=nums[i];
            }
        }
        return minIndex;
    }
}
```

