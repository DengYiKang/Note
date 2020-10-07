# Water and Jug Problem

### 问题

现有两个桶，两个桶的容量分别为x升和y升，要求量出z升的水。

### 解决方案：Bézout's identity，时间复杂度$O(logn)$，空间复杂度$O(1)$

Bézout's identity定理摘自wiki：

>Bézout's identity (also called Bézout's lemma) is a theorem in the elementary theory of numbers:
>
>let a and b be nonzero integers and let d be their greatest common divisor. Then there exist integers x
>and y such that ax+by=d
>
>In addition, the greatest common divisor d is the smallest positive integer that can be written as ax + by 
>
>every integer of the form ax + by is a multiple of the greatest common divisor d.

即当$z\%gcd(x,y)=0$时，$z=k_1*x+k_2*y$，其中$k_1,k_2$为整数。

```java
public boolean canMeasureWater(int x, int y, int z) {
    if(x + y < z) return false;
    if( x == z || y == z || x + y == z ) return true;
    return z%GCD(x, y) == 0;
}

public int GCD(int a, int b){
    while(b != 0 ){
        int temp = b;
        b = a%b;
        a = temp;
    }
    return a;
}
```

