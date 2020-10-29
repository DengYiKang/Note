# 4Sum II

### 问题

现有4个相同长度的数组`A,B,C,D`，求四元组`(i,j,k,l)`的种数，使得`A[i]+B[j]+C[k]+D[l]=0`。

### 解决方案

#### 滑动窗口，时间复杂度$O(n^3)$，空间复杂度$O(1)$

对`i,j`进行遍历，对于`k,l`可以采用滑动窗口的思想（见两数和）。

我认为对于求和问题，不管什么形式的数据（一个数组也好，多个数组也好），只要能找到一个策略，使得指针的变化有且只有一个选择才能使和变大或变小。

而对于数组`C,D`，先排序，后把指针`lo,hi`分别指向两数组的首位和末位，这种情况符合上述所讲的策略。

```java
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        Arrays.sort(C);
        Arrays.sort(D);
        int n=A.length;
        int ans=0;
        for(int i=0; i<n; i++){
            for(int j=0; j<n; j++){
                int target=-(A[i]+B[j]);
                int lo=0, hi=n-1, sum=0;
                while(lo<n&&hi>=0){
                    sum=C[lo]+D[hi];
                    if(sum==target){
                        int cCnt=1, dCnt=1;
                        while(lo+1<n&&C[lo]==C[lo+1]){
                            lo++;
                            cCnt++;
                        }
                        while(hi-1>=0&&D[hi]==D[hi-1]){
                            hi--;
                            dCnt++;
                        }
                        ans+=cCnt*dCnt;
                        lo++;
                        hi--;
                    }else if(sum>target){
                        hi--;
                    }else if(sum<target){
                        lo++;
                    }
                }
            }
        }
        return ans;
    }
}
```

#### 分组+保存每组和：时间复杂度$O(n^2)$，空间复杂度$O(n^2)$

```java
public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
    Map<Integer, Integer> map = new HashMap<>();
    
    for(int i=0; i<C.length; i++) {
        for(int j=0; j<D.length; j++) {
            int sum = C[i] + D[j];
            map.put(sum, map.getOrDefault(sum, 0) + 1);
        }
    }
    
    int res=0;
    for(int i=0; i<A.length; i++) {
        for(int j=0; j<B.length; j++) {
            res += map.getOrDefault(-1 * (A[i]+B[j]), 0);
        }
    }
    return res;
}
```

