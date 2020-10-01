# Bulb Switcher

### 问题

现有n个灯泡，初始状态均为熄灭。现循环n次，第i次把i的整数倍的位置的灯泡状态置反，求最终有多少灯泡亮着。

### 解决方案

对于第`i`次循环，考虑第`j`个灯泡，若`i>j`，则无影响。当`i<=j`时，考虑将`j`因式分解，注意总是成对地因式分解，最终若有$\sqrt{i}*\sqrt{i}=j$， 那么它的因子个数是奇数个，最终是亮的状态。因此这些亮灯泡的序数应该为`i^2`。

```java
class Solution {
    public int bulbSwitch(int n) {
        return (int) Math.sqrt(n);
    }
}
```

