# Maximum Swap

### 问题

将一个数的某两位进行交换，使得结果最大，求该结果。

### 解决方案

#### 一、

考虑哪些位可能是被换位（大的那个数）。易知对于某位`nums[i]`，如果有`nums[i-1]<nums[i]`，那么它可能是被换位。注意如果有`nums[i-1]<nums[i]=nums[i+1]`，那么被换位应该是`nums[i+1]`。

```java
class Solution {
    public int maximumSwap(int num) {
        int max_pos=-1;
        int tail=0;
        int[] nums=new int[10];
        int t_num=num;
        while(t_num>0){
            nums[tail++]=t_num%10;
            t_num/=10;
        }
        for(int i=tail-2; i>=0; i--){
            if(nums[i]>nums[i+1]){
                if(max_pos==-1||nums[i]>=nums[max_pos]) {
                    while(i>0&&nums[i]==nums[i-1]) i--;
                    max_pos=i;
                }
            }
        }
        if(max_pos==-1) return num;
        for(int i=tail-1; i>=0; i--){
            if(nums[i]<nums[max_pos]){
                int tmp=nums[i];
                nums[i]=nums[max_pos];
                nums[max_pos]=tmp;
                break;
            }
        }
        int ans=0;
        for(int i=tail-1; i>=0; i--){
            ans*=10;
            ans+=nums[i];
        }
        return ans;
    }
}
```

#### 二、贪心

结果越大=>大数排的越前即大数的最后出现的位置应该要靠前。

因此用`last[0...9]`数组表示`0~9`数字出现的最后位置，对`num`从左到右的扫描，如果存在大数的最后位置比当前位置靠后，则互换。

```java
class Solution {
    public int maximumSwap(int num) {
        char[] A=String.valueOf(num).toCharArray();
        int[] last=new int[10];
        for(int i=0; i<A.length; i++){
            last[A[i]-'0']=i;
        }
        for(int i=0; i<A.length; i++){
            for(int j=9; j>=0; j--){
                if(A[i]-'0'<j&&last[j]>i){
                    char tmp=A[i];
                    A[i]=A[last[j]];
                    A[last[j]]=tmp;
                    return Integer.valueOf(new String(A));
                }
            }
        }
        return num;
    }
}
```

