# 包含min函数的栈

### 问题

实现一个包含min函数的栈，要求各种操作的时间复杂度都为$O(1)$。

### 解决方案

对于压栈的数，如果小于当前的min值，那么将该数压入min栈；如果大于min值，则不做任何操作。

对于出栈的数，如果等于min栈的top，那么min栈也将top出栈。

```java
public class Solution {

    Stack<Integer> stack=new Stack<>();
    Stack<Integer> min=new Stack<>();
    
    public void push(int node) {
        stack.push(node);
        if(min.isEmpty()||node<=min.peek()){
            min.push(node);
        }
    }
    
    public void pop() {
        if(stack.isEmpty()) return;
        int node=stack.pop();
        if(node==min.peek()) min.pop();
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int min() {
        return min.peek();
    }
}
```

