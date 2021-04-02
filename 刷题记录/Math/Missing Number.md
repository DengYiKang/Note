# Missing Number

### 问题

给n个数，这n个数属于0~n，互不相同，找出0~n缺失的数。

### 解决方案：

#### 位运算

| Index | 0    | 1    | 2    | 3    |
| ----- | ---- | ---- | ---- | ---- |
| Value | 0    | 1    | 3    | 4    |

$Missing=4∧(0∧0)∧(1∧1)∧(2∧3)∧(3∧4)=(4∧4)∧(0∧0)∧(1∧1)∧(3∧3)∧2=0∧0∧0∧0∧2=2$

```java
class Solution {
    public int missingNumber(int[] nums) {
        int missing = nums.length;
        for (int i = 0; i < nums.length; i++) {
            missing ^= i ^ nums[i];
        }
        return missing;
    }
}
```

#### 高斯运算

$Missing=\Sigma_{i=0}^{n}i-\Sigma_i nums[i]$

```java
class Solution {
    public int missingNumber(int[] nums) {
        int expectedSum = nums.length*(nums.length + 1)/2;
        int actualSum = 0;
        for (int num : nums) actualSum += num;
        return expectedSum - actualSum;
    }
}
```



