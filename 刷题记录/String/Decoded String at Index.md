# Decoded String at Index

### 问题

`S`为编码的字符串，比如`xxx2`表示将`xxx`重复2次，`xxx23`表示将重复两次的`xxx`重复三次，即将`xxx`重复六次。`S`由小写字母和数字组成。`S`的首字符为小写字母。问第`K`个字母是什么。

### 解决方案：

#### 一、时间复杂度$O(n)$

用`s`表示字符串，用`x`表示数（已经将连续的数字进行累乘合并），则`S`可表示为：$x_0s_0x_1s_1x_2...$，其中$x_0$为0（方便后续计算）。

用`sum[i]`表示$x_0s_0x_1s_1...x_{i-1}s_{i-1}$的长度，`repeat[i]`表示$x_i$的大小,`s[i]`表示$s_i$这个字符串。

那么有：`sum[i]=sum[i-1]*repeat[i-1]+len(s[i-1])`。

对于当前的`i`，如果`pos>=sum[i]*repeat[i]`，那么`pos`对应的字符为`s[i].charAt(pos-sum[i]*repeat[i])`

否则将`pos`更新为`pos%sum[i]`，且将`i--`。

```java
class Solution {
    public String decodeAtIndex(String S, int K) {
        List<String> strList=new ArrayList<>();
        List<Long> repeat=new ArrayList<>();
        repeat.add((long)0);
        long product=1;
        StringBuilder sb=new StringBuilder();
        for(char ch:S.toCharArray()){
            if(ch>='a'&&ch<='z'){
                if(product!=1) repeat.add(product);
                sb.append(ch);
                product=1;
            }else{
                if(sb.length()!=0) strList.add(sb.toString());
                sb=new StringBuilder();
                product*=(ch-'0');
            }
        }
        if(product!=1) repeat.add(product);
        if(sb.length()!=0) strList.add(sb.toString());
        long[] sum=new long[repeat.size()];
        for(int i=1; i<repeat.size(); i++){
            sum[i]=sum[i-1]*repeat.get(i-1)+strList.get(i-1).length();
        }
        long pos=K-1;
        int i=repeat.size()-1;
        while(i>=0){
            if(pos>=sum[i]*repeat.get(i)){
                return String.valueOf(strList.get(i).charAt((int)(pos-sum[i]*repeat.get(i))));
            }else{
                pos=pos%sum[i];
                i--;
            }
        }
        return null;
    }
}
```

#### 二、时间复杂度$O(n)$

跟第一个方法的思路非常相似。

```java
class Solution {
    public String decodeAtIndex(String S, int K) {
        long size=0;
        for(char ch:S.toCharArray()){
            if(Character.isDigit(ch)) size*=ch-'0';
            else size++;
        }
        for(int i=S.length()-1; i>=0; i--){
            char ch=S.charAt(i);
            K%=size;
            if(K==0&&Character.isLetter(ch)) 
                return Character.toString(ch);
            if(Character.isDigit(ch))
                size/=ch-'0';
            else 
                size--;
        }
        return null;
    }
}
```

