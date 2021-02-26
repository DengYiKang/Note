# Validate Stack Sequences

### 问题

现有两个序列`pushed`和`popped`，分别表示同一个栈的push的顺序和pop的顺序，判断是否合法。

### 解决方案：时间复杂度$O(n)$ 

首先进行索引化，保证pushed数组是单调递增的。

从poped数组的角度遍历，如果当前popped[j]>=pushed[i]，那么将pushed[i] push。如果当前poped[j]<pushed[i]，判断栈顶是否为poped[j]，是则pop，继续；否则返回false。

```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
            if(pushed.length!=popped.length) return false;
            HashMap<Integer, Integer> mp=new HashMap<>();
            for(int i=0; i<pushed.length; i++){
                    mp.put(pushed[i], i+1);
                    pushed[i]=i+1;
            }
            for(int i=0; i<popped.length; i++){
                    popped[i]=mp.get(popped[i]);
            }
            int p=0;
            Stack<Integer> stack=new Stack<>();
            for(int i=0; i<popped.length; i++){
                    while(p<pushed.length&&popped[i]>=pushed[p]){
                            stack.push(pushed[p]);
                            p++;
                    }
                    if(stack.peek()!=popped[i]) return false;
                    stack.pop(); 
            }
            return true;
    }
}
```

