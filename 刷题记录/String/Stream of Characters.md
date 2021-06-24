# Stream of Characters

### 问题

给一个String[] words初始化，这些为模式串。然后输入query序列，如果在query序列中，以letter为尾的长度大于等于1的query序列如果在words中找得到，那么query(letter)返回true，否则返回false。注意，query(letter)是连续调用的。

```
StreamChecker streamChecker = new StreamChecker(["cd","f","kl"]); // init the dictionary.
streamChecker.query('a');          // return false
streamChecker.query('b');          // return false
streamChecker.query('c');          // return false
streamChecker.query('d');          // return true, because 'cd' is in the wordlist
streamChecker.query('e');          // return false
streamChecker.query('f');          // return true, because 'f' is in the wordlist
streamChecker.query('g');          // return false
streamChecker.query('h');          // return false
streamChecker.query('i');          // return false
streamChecker.query('j');          // return false
streamChecker.query('k');          // return false
streamChecker.query('l');          // return true, because 'kl' is in the wordlist
```

### 解决方案

这个可以用Trie解决，将word倒置，插入到Trie中，然后维护一个list来保存之前的query，然后从后到前进行查询即可。

但是也可以用AC自动机来做（虽然耗时更大）。

用AC自动机做，可以不需要记录之前的query，因为0->status这个路径表示当前已经匹配好了的串，新到来的query直接利用这个性质就好了，如果不匹配，不断地跳fail指针即可。检查当前query是否是true，可以不断跳fail指针，当存在一个fail指针指向的status的flag是true的，那么返回true。

> 这个题目也告诉我们AC自动机不仅可以用来匹配前缀（退化为Trie），也可以用来匹配后缀

```java
class StreamChecker {
    
    class Vertex{
        int[] next=new int[26];
        int[] go=new int[26];
        int link=-1;
        int pch='$';
        int p=-1;
        boolean flag=false;
        Vertex(int pch, int p){
            this.pch=pch;
            this.p=p;
            Arrays.fill(next, -1);
            Arrays.fill(go, -1);
        }
        Vertex(){
            Arrays.fill(next, -1);
            Arrays.fill(go, -1);
        }
    }
    List<Vertex> node=new ArrayList<>();
    int status=0;
    
    void add(String word){
        int v=0;
        for(int i=0; i<word.length(); i++){
            int id=word.charAt(i)-'a';
            if(node.get(v).next[id]==-1){
                node.get(v).next[id]=node.size();
                node.add(new Vertex(id, v));
            }
            v=node.get(v).next[id];
        }
        node.get(v).flag=true;
    }
    
    int get_link(int v){
        Vertex t=node.get(v);
        if(t.link!=-1) return t.link;
        if(v==0||t.p==0){
            t.link=0;
        }else{
            t.link=go(get_link(t.p), t.pch);
        }
        return t.link;
    }
    
    int go(int v, int ch){
        Vertex t=node.get(v);
        if(t.go[ch]!=-1) return t.go[ch];
        if(t.next[ch]!=-1){
            t.go[ch]=t.next[ch];
        }else{
            t.go[ch]=v==0?0:go(get_link(v), ch);
        }
        return t.go[ch];
    }
    
    
    public StreamChecker(String[] words) {
        status=0;
        node.clear();
        node.add(new Vertex());
        for(String word:words){
            add(word);
        }
    }
    
    public boolean query(char letter) {
        int id=letter-'a';
        while(node.get(status).next[id]==-1&&status!=0){
            status=get_link(status);
        }
        if(status==0){
            status=node.get(status).next[id]==-1?
                0:node.get(status).next[id];
        }else{
            status=node.get(status).next[id];
        }
        int tmp=status;
        while(tmp!=0){
            if(node.get(tmp).flag) return true;
            tmp=get_link(tmp);
        }
        return false;
    }

}
```

