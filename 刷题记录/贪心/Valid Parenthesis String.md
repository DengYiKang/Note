# Valid Parenthesis String

### 问题

一个字符串由`'(', ')'. '*'`三种字符组成，其中`*`可以表示成左括号或右括号或空值。求问该表达式是否合法的？

### 解决方案：

`*`最后考虑取值，优先使用左括号与右括号相抵消，最后再考虑`*`。

#### 一、stack，时间复杂度$o(n), O(n^2)$

优先使用左括号与右括号相抵消，即当把所有顶端的`*`弹出，将第一个弹出的左括号与右括号抵消，如果没有左括号，则将`*`的数量减一，最后将弹出的`*`压会栈中。

最后栈中只剩`*`与`(`，用另一个栈来判断是否合法。

```java
class Solution {
    public boolean checkValidString(String s) {
        Stack<Character> stack=new Stack<>();
        char[] chs=s.toCharArray();
        for(int i=0; i<chs.length; i++){
            if(chs[i]=='*'||chs[i]=='('){
                stack.push(chs[i]);
            }else{
                if(stack.isEmpty()) return false;
                else{
                    int cnt=0;
                    while(!stack.isEmpty()&&stack.peek()=='*'){
                        stack.pop();
                        cnt++;
                    }
                    if(stack.isEmpty()){
                        if(cnt==0) return false;
                        else cnt--;
                    }else{
                        if(stack.peek()=='(') stack.pop();
                        else{
                            if(cnt==0) return false;
                            else cnt--;
                        }
                    }
                    while(cnt-->0){
                        stack.push('*');
                    }
                }
            }
        }
        Stack<Character> stack2=new Stack<>();
        while(!stack.isEmpty()){
            char ch=stack.pop();
            if(ch=='*') stack2.push(ch);
            else{
                if(stack2.isEmpty()) return false;
                else stack2.pop();
            }
        }
        return true;
    }
}
```

#### 二、贪心，时间复杂度$O(n)$

因为`*`代表任意值，因此可以计算左括号的最小剩余次数`lo`以及最大剩余次数`hi`。

在遍历的过程中，如果`hi<0`，那必然是非法的（`hi`这种情况是`*`全取左括号的情况）。

到最后，如果`lo==0`，那么是合法的。

```java
class Solution {
    public boolean checkValidString(String s) {
       int lo = 0, hi = 0;
       for (char c: s.toCharArray()) {
           lo += c == '(' ? 1 : -1;
           hi += c != ')' ? 1 : -1;
           if (hi < 0) return false;
           lo = Math.max(lo, 0);
       }
       return lo == 0;
    }
}
```

