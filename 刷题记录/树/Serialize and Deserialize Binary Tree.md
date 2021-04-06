# Serialize and Deserialize Binary Tree

### 问题

对树的序列化和反序列化。

### 解决方案：

```java
public class Codec {

    public String serialize(TreeNode root) {
        Queue<TreeNode> queue=new LinkedList<>();
        queue.offer(root);
        StringBuilder sb=new StringBuilder();
        if(root==null) return "";
        sb.append(root.val);
        sb.append(",");
        while(!queue.isEmpty()){
            TreeNode node=queue.poll();
            if(node==null) continue;
            // if(node.left!=null) queue.offer(node.left);
            // else sb.append("-10000,");
            // if(node.right!=null) queue.offer(node.right);
            // else sb.append("-10000,");
            int val=node.left==null?-10000:node.left.val;
            sb.append(String.valueOf(val)+",");
            queue.offer(node.left);
            val=node.right==null?-10000:node.right.val;
            sb.append(String.valueOf(val)+",");
            queue.offer(node.right);
        }
        return sb.toString();
    }

    public TreeNode deserialize(String data) {
        if(data==null||data.length()==0) return null;
        String[] splits=data.split(",");
        Queue<TreeNode> queue=new LinkedList<>();
        TreeNode root=new TreeNode(Integer.valueOf(splits[0]));
        queue.offer(root);
        int tail=1;
        while(!queue.isEmpty()){
            TreeNode node=queue.poll();
            int val=Integer.valueOf(splits[tail++]);
            if(val!=-10000){
                node.left=new TreeNode(val);
                queue.offer(node.left);
            }
            val=Integer.valueOf(splits[tail++]);
            if(val!=-10000){
                node.right=new TreeNode(val);
                queue.offer(node.right);
            }
        }
        return root;
    }
}
```

