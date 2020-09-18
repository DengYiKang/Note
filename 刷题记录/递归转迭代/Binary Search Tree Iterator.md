# Binary Search Tree Iterator

### 问题

实现一个BST的迭代器，要求next()与hasNext()函数平均耗时$O(1)$

### 解决方案

+ 可以先把BST遍历一遍，将结果存起来，需要时即取

  ```java
  class BSTIterator {
      
      private List<Integer> list;
      private Deque<TreeNode> stack;
      private TreeNode p;
      private int index;
  
      public BSTIterator(TreeNode root) {
          list=new ArrayList<>();
          stack=new ArrayDeque<>();
          TreeNode p=root;
          while(p!=null||!stack.isEmpty()){
              if(p!=null){
                  stack.push(p);
                  p=p.left;
              }else{
                  p=stack.pop();
                  list.add(p.val);
                  p=p.right;
              }
          }
          index=0;
      }
      
      /** @return the next smallest number */
      public int next() {
          return list.get(index++);
      }
      
      /** @return whether we have a next smallest number */
      public boolean hasNext() {
          return index<list.size();
      }
  }
  ```

+ 即时获取

  ```java
  class BSTIterator {
      
      private Deque<TreeNode> stack;
  
      public BSTIterator(TreeNode root) {
          stack=new ArrayDeque<>();
          if(root!=null) {
              stack.push(root);
              pushAll(root.left);
          }
      }
      
      /** @return the next smallest number */
      public int next() {
          TreeNode node=stack.pop();
          pushAll(node.right);
          return node.val;
      }
      
      /** @return whether we have a next smallest number */
      public boolean hasNext() {
          return !stack.isEmpty();
      }
      
      public void pushAll(TreeNode root){
          while(root!=null){
              stack.push(root);
              root=root.left;
          }
      }
  }
  ```

  