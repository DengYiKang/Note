# Different Ways to Add Parentheses

### 问题

输入一串的数字与+，-，*混合的合法的表达式，现向该表达式插入任意多组合法的括号，要求输出所有的可能的计算结果。

### 解决方案

例如，如果表达式为`2*3-4*5`，现有一种方案为`2*(3-(4*5))`，那么可以用树来表示：

<img src="/home/yikang/Documents/gitFiles/Note/刷题记录/pic/4.png" alt="4" style="zoom: 80%;" />

注意到所有方案对应的异构BST，它们的前中后序遍历都是一致的，中序遍历即为表达式，因此可以只构造操作符的BST，对于操作数可以递归递增访问数组取得。

```java
class Solution {
    
    class Node{
        int val;
        Node left;
        Node right;
        Node(int val){
            this.val=val;
        }
    }
    
    //构造所有的异构BST
    List<Node> build(int lo, int hi){
        List<Node> trees=new ArrayList<>();
        if(lo>hi){
            trees.add(null);
            return trees;
        }
        for(int mid=lo; mid<=hi; mid++){
            List<Node> left=build(lo, mid-1);
            List<Node> right=build(mid+1, hi);
            for(int i=0; i<left.size(); i++){
                for(int j=0; j<right.size(); j++){
                    Node root=new Node(mid);
                    root.left=left.get(i);
                    root.right=right.get(j);
                    trees.add(root);
                }
            }
        }
        return trees;
    }
    
    //计算root结点的结果
    int cal(Node root){
        if(root==null) return nums.get(num_cnt++);
        int left=cal(root.left);
        int right=cal(root.right);
        char op=mp.get(root.val);
        if(op=='+'){
            return left+right;
        }else if(op=='-'){
            return left-right;
        }else {
            return left*right;
        }
    }
    
    public List<Integer> diffWaysToCompute(String input) {
        mp=new HashMap<>();
        nums=new ArrayList<>();
        op_cnt=0;
        List<Integer> ans=new ArrayList<>();
        if(input.length()==0) return ans;
        int lo, hi;
        lo=hi=0;
        for(int i=0; i<input.length(); i++){
            char ch=input.charAt(i);
            if(ch=='+'||ch=='-'||ch=='*'){
                if(lo!=hi) nums.add(Integer.valueOf(input.substring(lo, hi)));
                lo=hi=i+1;
                mp.put(++op_cnt, ch);
            }else{
                hi++;
            }
        }
        nums.add(Integer.valueOf(input.substring(lo, hi)));
        if(nums.size()==1) return nums;
        List<Node> trees=build(1, op_cnt);
        for(Node root:trees){
            num_cnt=0;
            ans.add(cal(root));
        }
        return ans;
    }
    
    int op_cnt;
    int num_cnt;
    HashMap<Integer, Character> mp;
    List<Integer> nums;
    
}
```

### 补充

上述需要建树，占用大量的空间，可以考虑用dp的方式：

```java
class Solution {
    public List<Integer> diffWaysToCompute(String expression) {
        String[] nums=expression.split("\\*|-|\\+");
        String[] tmp=expression.split("\\d+");
        if(tmp.length<2){
            List<Integer> ans=new ArrayList<>();
            ans.add(Integer.valueOf(nums[0]));
            return ans;
        }
        String[] ops=Arrays.copyOfRange(tmp, 1, tmp.length);
        return cal(nums, ops, 0, nums.length+ops.length-1);
    }
    List<Integer> cal(String[] nums, String[] ops, int l, int r){
        List<Integer> ans=new ArrayList<>();
        if(l==r){
            ans.add(Integer.valueOf(nums[l/2]));
            return ans;
        } 
        for(int i=l+1; i<r; i+=2){
            List<Integer> left_nums=cal(nums, ops, l, i-1);
            List<Integer> right_nums=cal(nums, ops, i+1, r);
            for(int left:left_nums){
                for(int right:right_nums){
                    if(ops[i/2].equals("*")) 
                        ans.add(left*right);
                    else if(ops[i/2].equals("+"))
                        ans.add(left+right);
                    else 
                        ans.add(left-right);
                }
            }
        }
        return ans;
    }
}
```

