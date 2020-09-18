# Fraction to Recurring Decimal

### 问题

输入分子与分母，均为整型，要求输出结果，小数的循环部分用括号括起来，如0.(012)。

### 解决方案

把各位的余数用map存起来，当余数与之前出现过的余数有重复时即可确定循环小数部分

```java
class Solution {
    public String fractionToDecimal(int numerator, int denominator) {
        if(numerator==0) return "0";
        StringBuilder ans=new StringBuilder();
        HashMap<Long, Integer> mp=new HashMap<>();
        ans.append((numerator>0)^(denominator>0)?"-":"");
        long a=Math.abs((long) numerator);
        long b=Math.abs((long) denominator);
        ans.append(a/b);
        a=a%b;
        if(a==0) return ans.toString();
        ans.append(".");
        mp.put(a, ans.length());
        while(a!=0){
            a*=10;
            ans.append(a/b);
            a=a%b;
            if(mp.containsKey(a)){
                ans.insert(mp.get(a), "(");
                ans.append(")");
                break;
            }else{
                mp.put(a, ans.length());
            }
        }
        return ans.toString();
    }
}
```

