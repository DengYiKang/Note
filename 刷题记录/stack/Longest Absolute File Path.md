## Longest Absolute File Path

### 问题

输入一个字符串，例如：

`"dir\n\tsubdir1\n\t\tfile1.ext\n\t\tsubsubdir1\n\tsubdir2\n\t\tsubsubdir2\n\t\t\tfile2.ext"`

![img](../pic/8.jpg)

找到拥有最长绝对路径的文件，输出其绝对路径长度。

### 解决方案

+ 定义结点，存储父结点，`\t`的数量表示深度，根据当前深度和当前字符串中`\t`的数量决定要不要回溯父结点

```java
    class Node{
        int d=0;
        Node parent=null;
        int height=0;
        Node(){}
        Node(int d, Node parent, int height){
            this.d=d;
            this.parent=parent;
            this.height=height;
        }
    }
    public int lengthLongestPath(String input) {
        int ans=0;
        if(input==null||input.length()==0) return 0;
        String[] splits=input.split("\n");
        if(splits==null||splits.length==0) return 0;
        if(splits[0].indexOf(".")!=-1){
            return splits[0].length();
        }
        Node root=new Node(splits[0].length(), null, 0);
        Node cur=root;
        for(int i=1; i<splits.length; i++){
            Node tmp=new Node();
            StringBuilder sb=new StringBuilder(splits[i]);
            int h=sb.lastIndexOf("\t")+1;
            while(cur.height+1!=h&&cur.parent!=null) cur=cur.parent;
            tmp.height=h;
            tmp.parent=cur;
            tmp.d=tmp.parent.d+sb.length()+1-h;
            if(sb.indexOf(".")!=-1) ans=Math.max(ans, tmp.d);
            cur=tmp;
        }
        return ans;
    }
```

+ 该题符合栈的特点

  ```java
  class Solution {
      public int lengthLongestPath(String input) {
          Stack<Integer> stack=new Stack<>();
          int ans=0;
          stack.push(-1);
          for(String s:input.split("\n")){
              int h=s.lastIndexOf("\t");
              //巧妙地利用stack.size
              while(h+2!=stack.size()) stack.pop();
              int len=stack.peek()+s.length()-h;
              if(s.indexOf(".")!=-1) ans=Math.max(ans, len);
              else stack.push(len);
          }
          return ans;
      }
  }
  ```

  