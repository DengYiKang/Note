# Ambiguous Coordinates

### 问题

给一串数字，要求把它划分成两个数值，可对两个数值加上小数点`.`（也可以不加），要求两个数值是合法的数，且是最简表示。即不能是`00`， `0.00`，`1.00`，`01`等等。

### 解决方案：时间复杂度$O(n^3)$

这题主要的难度在于加小数点使得生成的数值符合题意这块。

将小数点视作分隔符，那么就有左右两个字符串`left`和`right`，那么只需要判断以下条件：

`!left.startsWith("0")||left.eauqls("0")&&!right.endsWith("0")`。

至于不加小数点的情况，就是`right`为空串这一情况。

```java
class Solution {
    public List<String> ambiguousCoordinates(String S) {
        List<String> ans=new ArrayList<>();
        for(int i=2; i<S.length()-1; i++){
            for(String left:make(S, 1, i)){
                for(String right:make(S, i, S.length()-1)){
                    ans.add("("+left+", "+right+")");
                }
            }
        }
        return ans;
    }
    public List<String> make(String s, int lo, int hi){
        List<String> res=new ArrayList<>();
        for(int mid=lo+1; mid<=hi; mid++){
            String left=s.substring(lo, mid);
            String right=s.substring(mid, hi);
            if((!left.startsWith("0")||left.equals("0"))
               &&!right.endsWith("0")){
                res.add(left+(mid==hi?"":".")+right);
            }
        }
        return res;
    }
}
```

