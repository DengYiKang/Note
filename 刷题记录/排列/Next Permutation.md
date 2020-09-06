# Next Permutation

### 问题

输入一个整数排列，输出下一个大的排列（字典序），若字典序已为最大值，则输出字典序最小的排列。

### 解决方案

+ 当排列是降值排列时，没有更大的排列。
+ 当某段后缀是先升值后降值时，如3**4**5321，将4与序列[5,3,2,1]中比4大的最小值与4替换，再将后续数列从小到大排序。
+ 当某段后缀是升值排列时，交换两数。
+ 处理顺序从后往前

```java
    public void nextPermutation(int[] nums) {
        if(nums.length<2) return;
        int diff=0;
        for(int i=nums.length-2; i>=0; i--){
            int new_diff=0;
            if(i==nums.length-2) new_diff=diff=nums[i]-nums[i+1];
            else new_diff=nums[i]-nums[i+1];
            //第二种情况
            if(diff>=0&&new_diff<0){
                int tmp=nums[i], index=find_index(i, nums);
                nums[i]=nums[index];
                nums[index]=tmp;
                Arrays.sort(nums, i+1, nums.length);
                return;
            }else if(diff<0){
                //第三种情况
                int tmp=nums[i];
                nums[i]=nums[i+1];
                nums[i+1]=tmp;
                return;
            }
        }
        Arrays.sort(nums);
    }
	//在nums[index...len-1]中寻找比nums[index]大的最小值的索引
    public int find_index(int index, int[] nums){
        int min_val=Integer.MAX_VALUE, ans=index;
        for(int i=index+1; i<nums.length; i++){
            if(nums[i]>nums[index]&&min_val>nums[i]){
                min_val=nums[i];
                ans=i;
            }
        }
        return ans;
    }
```

