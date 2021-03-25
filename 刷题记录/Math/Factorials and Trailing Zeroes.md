# Factorials and Trailing Zeroes

### 问题

对于一个数$n!$。求它末尾连续0的个数。

### 解决方案：时间复杂度$O(logn)$

找到10的因子的个数，10=2*5，又因为偶数远远比末尾为5的个数大，因此只需要考虑1~n!中以5结尾的数的数量。

${n！}/5$ 会遗漏，如果当前数是25、125等等，他们可以被因式分解为5\*5、5\*5\*5，因此将$n!$除以5之后还需要再次除以5，直到除尽为止。

```java
public int trailingZeroes(int n) {
    int count = 0;

    while(n > 0) {
        n /= 5;
        count += n;
    }

    return count;
}
```



