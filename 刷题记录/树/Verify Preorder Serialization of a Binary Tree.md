# Verify Preorder Serialization of a Binary Tree

### 问题

输入由`,`分割的字符串，其中数字表示结点，`#`表示`null`的哨兵，求问该字符串是否是某BST树的合法前序遍历结果

### 解决方案

因为是前序，所以可以很好地用迭代方式解决。因为是前序，所以要先生成遇见的结点。

对于一个输入字符，不管是数字或是哨兵，若有子树空位，则生成结点（哨兵可不生成），若无子树空位，则回退至上一个结点。

```java
class Solution {
    public boolean isValidSerialization(String preorder) {
        if(preorder==null||preorder.length()==0) return true;
        Stack<boolean[]> stack=new Stack<>();
        String[] splits=preorder.split(",");
        if(!splits[0].equals("#")) stack.push(new boolean[]{false, false});
        int tail=1;
        while(!stack.isEmpty()&&tail<splits.length){
            boolean[] parent=stack.pop();
            if(parent[0]==false) parent[0]=true;
            else if(parent[1]==false) parent[1]=true;
            else{
                continue;
            }
            stack.push(parent);
            if(!splits[tail++].equals("#")){
                stack.push(new boolean[]{false, false});
            }
        }
        while(!stack.isEmpty()&&stack.peek()[0]==true&&stack.peek()[1]==true){
            stack.pop();
        }
        if(!stack.isEmpty()||tail<splits.length) return false;
        return true;
    }
}
```

