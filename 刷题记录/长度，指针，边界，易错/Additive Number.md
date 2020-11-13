# Additive Number

### 问题

输入一个字符串`num`，求能否把`num`切割成三块以上，使得前两块的和等于下一块。注意每块不能前置0，比如`1, 2, 03` 和 `1, 02, 3` 是非法的，但`0,0,1,1`是合法的。

### 解决方案

从长度的角度出发，结合`substring`函数更佳。

```java
import java.math.BigInteger;

class Solution {
    public boolean isAdditiveNumber(String num) {
        for(int i=1; i<num.length()/2+1; i++){
            if(num.charAt(0)=='0'&&i>1) return false;
            BigInteger x1=new BigInteger(num.substring(0, i));
            for(int j=1; Math.max(i, j)<=num.length()-i-j; j++){
                if(num.charAt(i)=='0'&&j>1) break;
                BigInteger x2=new BigInteger(num.substring(i, i+j));
                if(valid(x1, x2, i+j, num)) return true;
            }  
        }
        return false;
    }
    boolean valid(BigInteger x1, BigInteger x2, int start, String num){
        if(start==num.length()) return true;
        x2=x2.add(x1);
        x1=x2.subtract(x1);
        String sum=x2.toString();
        return num.startsWith(sum, start)&&valid(x1 ,x2, start+sum.length(), num);
    }
}
```

以上是尾递归的形式，可以化成迭代的形式：

```java
public class Solution {
    public boolean isAdditiveNumber(String num) {
        int n = num.length();
        for (int i = 1; i <= n / 2; ++i)
            for (int j = 1; Math.max(j, i) <= n - i - j; ++j)
                if (isValid(i, j, num)) return true;
        return false;
    }
    private boolean isValid(int i, int j, String num) {
        if (num.charAt(0) == '0' && i > 1) return false;
        if (num.charAt(i) == '0' && j > 1) return false;
        String sum;
        BigInteger x1 = new BigInteger(num.substring(0, i));
        BigInteger x2 = new BigInteger(num.substring(i, i + j));
        for (int start = i + j; start != num.length(); start += sum.length()) {
            x2 = x2.add(x1);
            x1 = x2.subtract(x1);
            sum = x2.toString();
            if (!num.startsWith(sum, start)) return false;
        }
        return true;
    }
}
```

