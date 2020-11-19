# Optimal Division

### 问题

对于一串表达式，如`2/3/3`，在其中加入任意多的成对括号，要求表达式的值最大，返回没有重复的成对括号的最大值表达式。

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

考虑`a/b/c/d`，要使其值最大，那么先使`b/c/d`最小。

注意到：

```shell
b/(c/d)        (b/c)/d = b/c/d
(b*d)/c        b/(d*c)
d/c            1/(d*c)
```

显然有`b/c/d<b/(c/d)`。

对于`a`，如果有形如`(expr1)/(expr2)`的表达式，可先使`expr2`最小，再从`expr1`中剥离出`(expr)/(expr)`的形式，不断进行递归处理，因此最后分子必然是一个独立的数字，而不是括号。

```java
class Solution {
    public String optimalDivision(int[] nums) {
        if(nums.length==1) return nums[0]+0;
        if(nums.length==2) return nums[0]+"/"+nums[1];
        StringBuilder sb=new StringBuilder(nums[0]+"/("+nums[1]);
        for(int i=2; i<nums.length; i++){
            sb.append("/");
            sb.append(nums[i]);
        }
        sb.append(")");
        return sb.toString();
    }
}
```



