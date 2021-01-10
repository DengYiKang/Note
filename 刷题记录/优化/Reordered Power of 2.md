# Reordered Power of 2

### 问题

一个正的整型数组`N`，可以将`N`的所有数字（十进制）重组，判断重组后的数（包含本身，首位不为0）是否是2的次方倍。

### 解决方案

#### 一、排列，时间复杂度$O((logn)!*logn)$

```java
class Solution {
    public boolean reorderedPowerOf2(int N) {
        List<int[]> list=new ArrayList<>();
        int[] nums=new int[9];
        int tail=0;
        while(N>0){
            nums[tail++]=N%10;
            N/=10;
        }
        //得到排列
        permutations(list, Arrays.copyOfRange(nums, 0, tail), 0, new boolean[tail]);
        for(int[] pos:list){
            long sum=0;
            if(nums[pos[0]]==0) continue;
            for(int i=0; i<pos.length; i++){
                sum*=10;
                sum+=nums[pos[i]];
            }
            int cnt=0;
            while(sum>0){
                cnt+=sum&1;
                if(cnt>1) break;
                sum>>=1;
            }
            if(cnt==1) return true;
        }
        return false;
    }
    public void permutations(List<int[]> list, int[] order, int pos, boolean[] vis){
        if(pos==order.length){
            list.add(Arrays.copyOf(order, order.length));
        }
        for(int i=0; i<order.length; i++){
            if(vis[i]) continue;
            order[pos]=i;
            vis[i]=true;
            permutations(list, order, pos+1, vis);
            vis[i]=false;
        }
    }
}
```

#### 二、count，时间复杂度$O(logn)$

在正整型范围内，只有31个数是2的次方倍（即首位是1）。

只需判断`N`是否能重组成这31个数之一就行了。

```java
class Solution {
    public boolean reorderedPowerOf2(int N) {
        int[] A=count(N);
        for(int i=0; i<31; i++){
            if(Arrays.equals(A, count(1<<i))) return true;
        }
        return false;
    }
    public int[] count(int N){
        int[] ans=new int[10];
        while(N>0){
            ans[N%10]++;
            N/=10;
        }
        return ans;
    }
}
```

