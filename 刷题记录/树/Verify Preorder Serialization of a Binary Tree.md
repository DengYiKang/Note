# Verify Preorder Serialization of a Binary Tree

### 问题

输入由`,`分割的字符串，其中数字表示结点，`#`表示`null`的哨兵，求问该字符串是否是某BST树的合法前序遍历结果

### 解决方案

因为是前序，所以可以很好地用迭代方式解决。因为是前序，所以要先生成遇见的结点。

对于一个输入字符，不管是数字或是哨兵，若有子树空位，则生成结点（哨兵可不生成），若无子树空位，则回退至上一个结点。

```java
class Solution {
    class Node{
        boolean l=false;
        boolean r=false;
        Node(){}
        Node(boolean l, boolean r){
            this.l=l;
            this.r=r;
        }
    }
    public boolean isValidSerialization(String preorder) {
        if(preorder==null||preorder.length()==0) return false;
        String[] splits=preorder.split(",");
        int cnt=1;
        Stack<Node> stack=new Stack<>();
        if(splits[0].equals("#")) stack.push(new Node(true, true));
        else stack.push(new Node());
        while(!stack.isEmpty()){
            Node cur=stack.peek();
            //这里对结点同一处理，保证在栈非空的情况下当前结点有子树空位可用
            while(!stack.isEmpty()&&cur.l&&cur.r){
                stack.pop();
                if(!stack.isEmpty()) cur=stack.peek();
            }
            //栈中结点不够用
            if(cur.l&&cur.r) break;
            //输入的结点不够用
            if(cnt==splits.length) return false;
            if(!splits[cnt].equals("#")){
                if(!cur.l){
                    cur.l=true;
                    cnt++;
                    stack.push(new Node());
                }else if(!cur.r){
                    cur.r=true;
                    cnt++;
                    stack.push(new Node());
                }
            }else{
                if(!cur.l) cur.l=true;
                else if(!cur.r) cur.r=true;
                cnt++;
            }
        }
        if(cnt!=splits.length) return false;
        return true;
    }
}
```

