# Divide Two Integers

### 问题

不能使用乘除模运算符，实现整数除法，返回商，小数向0取整，例如truncate(8.345) = 8；truncate(-2.7335) = -2。若溢出则返回$2^{31}-1$。

### 解决方案：$O(({logN})^2)$

```java
public int divide(int A, int B) {
        if (A == 1 << 31 && B == -1) return (1 << 31) - 1;
        int a = Math.abs(A), b = Math.abs(B), res = 0, x = 0;
        while (a - b >= 0) {
            for (x = 0; a - (b << x << 1) >= 0; x++);
            res += 1 << x;
            a -= b << x;
        }
        return (A > 0) == (B > 0) ? res : -res;
}
```

