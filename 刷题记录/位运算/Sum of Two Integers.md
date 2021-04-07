# Sum of Two Integers

### 问题

求两数和，不能使用+与-。

### 解决方案：位运算，时间复杂度$O(1)$

当前位是否为一取决于三个数(a，b，c)中的1的个数是奇数还是偶数，可以使用异或。

是否有进位取决于三个数中为1的个数是否大于等于2。

```java
class Solution {
    public int getSum(int a, int b) {
        int ans=0, c=0;
        for(int i=0; i<32; i++){
            int a_bit=(a>>>i)&1;
            int b_bit=(b>>>i)&1;
            boolean isOne=false;
            boolean[] hasC=new boolean[2];
            if(a_bit==1) {
                isOne=!isOne;
                hasC[0]=true;
            }
            if(b_bit==1) {
                isOne=!isOne;
                if(!hasC[0]) hasC[0]=true;
                else hasC[1]=true;
            }
            if(c==1) {
                isOne=!isOne;
                if(!hasC[0]) hasC[0]=true;
                else hasC[1]=true;
            }
            int cur=isOne?1:0;
            c=hasC[1]?1:0;
            ans=ans|(cur<<i);
        }
        return ans;
    }
}
```

