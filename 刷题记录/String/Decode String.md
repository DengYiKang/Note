# Decode String

### 问题

解析字符串。

**Example 1:**

```
Input: s = "3[a]2[bc]"
Output: "aaabcbc"
```

**Example 2:**

```
Input: s = "3[a2[c]]"
Output: "accaccacc"
```

**Example 3:**

```
Input: s = "2[abc]3[cd]ef"
Output: "abcabccdcdcdef"
```

**Example 4:**

```
Input: s = "abc3[cd]xyz"
Output: "abccdcdcdxyz"
```

### 解决方案：时间复杂度$O(n)$

下面这段代码虽然过了，但写的太蠢了。。。

```java
//递归思想
class Solution {
    
    public String decodeString(String s) {
        if(s==null||s.length()==0) return "";
        return decode(0, s.length(), s, 1);
    }
    public String decode(int start, int end, String s, int repeat){
        if(start>=end) return "";
        StringBuilder sb=new StringBuilder();
        int left=s.indexOf('[', start);
        if(left==-1||left>=end){
            for(int i=0; i<repeat; i++){
                sb.append(s.substring(start, end));
            }
            return sb.toString();
        }
        int cur=start, pre=start;
        while(cur<end){
            int[] pair=findFirstBracket(cur, end, s);
            if(pair==null){
                sb.append(s.substring(cur, end));
                break;
            }
            int[] numPair=findNum(pair[0], s);
            sb.append(s.substring(cur, numPair[0]));
            //递归调用
            sb.append(decode(pair[0]+1, pair[1], s, numPair[1]));
            cur=pair[1]+1;
        }
        String pattern=sb.toString();
        for(int i=1; i<repeat; i++) sb.append(pattern);
        return sb.toString();
    }
    boolean isDigit(char ch){
        int id=ch-'0';
        return id>=0&&id<=9;
    }
    //找到repeat数
    int[] findNum(int right, String s){
        int pos=right-1;
        while(pos>=0&&isDigit(s.charAt(pos))) pos--;
        return new int[]{pos+1, Integer.parseInt(s.substring(pos+1, right))};
    }
    //找到第一个Bracket的两端
    int[] findFirstBracket(int start, int end, String s){
        int left=s.indexOf('[', start);
        if(left==-1||left>=end) return null;
        int sum=1;
        int cur=left+1;
        while(cur<end){
            if(s.charAt(cur)=='[') sum++;
            else if(s.charAt(cur)==']') {
                sum--;
                if(sum==0) return new int[]{left, cur};
            }
            cur++;
        }
        return null;
    }
}
```

stack方法：

```java
    public String decodeString(String s) {
        Stack<Integer> intStack = new Stack<>();
        Stack<StringBuilder> strStack = new Stack<>();
        StringBuilder cur = new StringBuilder();
        int k = 0;
        for (char ch : s.toCharArray()) {
            if (Character.isDigit(ch)) {
                //这个计算方法好
                k = k * 10 + ch - '0';
            } else if ( ch == '[') {
                intStack.push(k);
                strStack.push(cur);
                cur = new StringBuilder();
                k = 0;
            } else if (ch == ']') {
                StringBuilder tmp = cur;
                cur = strStack.pop();
                for (k = intStack.pop(); k > 0; --k) cur.append(tmp);
            } else cur.append(ch);
        }
        return cur.toString();
    }
```

