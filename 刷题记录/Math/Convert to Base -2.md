# Convert to Base -2

### 问题

对给定的整数`N`，要求输出以-2为底的二进制串。

### 解决方案：时间复杂度$O(1)$

```java
  public String baseNeg2(int N) {
        StringBuilder res = new StringBuilder();
        while (N != 0) {
            res.append(N & 1);
            N = -(N >> 1);
        }
        return res.length() > 0 ? res.reverse().toString() : "0";
    }
```