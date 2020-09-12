# Gray Code

### 问题

输入一个正整数n，要求输出个数为1<<n的数组，其相邻的两数的二进制之间只有一位不同。

### 解决方案

感觉只要符合要求，不断变换，总能把1<<n个数全部枚举出来，所以当时用了dfs。

更好的代码如下，还没弄懂i ^ (i >>1)是怎么来的:

```java
class Solution {
    public List<Integer> grayCode(int n) {
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < (1 << n); i++) {
            res.add(i ^ (i >>1));
        }
        
        return res;
    }
}
```

