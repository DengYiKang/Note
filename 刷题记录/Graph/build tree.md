# 建树

```java
//给中序以及后序，要求输出前序
class Node{
    public int data;
    public Node left, right;
    Node(int x){
        data=x;
        left=right=null;
    }
};
int[] in=new int[maxn];//中序
int[] post=new int[maxn];//后序
int n, cnt;
int find_root(int x){
    for(int i=0; i<n; i++)
        if(in[i]==x) return i;
    return -1;
}
void build(Node root, int left, int right){
    if(right<left||cnt<0){
        root=null;
        return;
    }
    root=new Node(post[cnt]);
    int index=find_root(post[cnt]);
    cnt--;
    build(root.left, left, index-1);
    build(root.right, index+1, right);
}
void postOrder(Node t){
    if(t) return;
    System.out.print(t.data);
    postOrder(t.left);
    postOrder(t.right);
}
void init(){
    Scanner sc=New Scanner("...");
    for(int i=0; i<n; i++) in[i]=sc.nextInt();
    for(int i=0; i<n; i++) post[i]=sc.nextInt();
    Node root=null;
    cnt=n-1;
    build(root, 0, n-1);
    preorder(root);
}
```

