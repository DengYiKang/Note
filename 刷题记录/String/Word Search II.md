# Word Search II

### 问题

给一个词典String[] words，一个board，要求在board上找到词典中的词，这些块必须是相邻的序列（即能从起点出发不重复地走完），每个块最多使用一次，问词典中的词能在board上找到的词。n为词典的数量，m为board的总块数。

词典的数量很大，且词的最大长度不超过10。board的最大块数不超过12*12。

### 解决方案：Trie，时间复杂度$O(nm)$

由题可知，词典中的词的重叠数非常大，因此使用Trie来存。

然后对board进行dfs的过程中，利用Trie.next[id]==null来剪枝。

```java
class Solution {
    int[] step_x={1,0,-1,0};
    int[] step_y={0,1,0,-1};
    
    class Trie{
        Trie[] next=new Trie[26];
        String word=null;
    }
    
    public List<String> findWords(char[][] board, String[] words) {
        Trie root=new Trie();
        build(root, words);
        boolean[][] vis=new boolean[board.length][board[0].length];
        List<String> ans=new ArrayList<>();
        for(int i=0; i<board.length; i++){
            for(int j=0; j<board[0].length; j++){
                dfs(i, j, board, vis, ans, root);
            }
        }
        return ans;
    }
    
    public void build(Trie root, String[] list){
        for(int j=0; j<list.length; j++){
            Trie node=root;
            for(int i=0; i<list[j].length(); i++){
                int id=list[j].charAt(i)-'a';
                if(node.next[id]==null) node.next[id]=new Trie();
                node=node.next[id];
            }
            node.word=list[j];
        }
    }
    
    public void dfs(int x, int y, char[][] board, boolean[][] vis, List<String> ans, Trie node){
        if(!isValid(x, y, board, vis)) return;
        int id=board[x][y]-'a';
        if(node.next[id]==null) return;
        node=node.next[id];
        vis[x][y]=true;
        if(node.word!=null) {
            ans.add(node.word);
            node.word=null;
        }
        for(int i=0; i<4; i++){
            int nx=x+step_x[i];
            int ny=y+step_y[i];
            dfs(nx, ny, board, vis, ans, node);
        }
        vis[x][y]=false;
    }
    boolean isValid(int x, int y, char[][] board, boolean[][] vis){
        if(x>=board.length||x<0) return false;
        if(y>=board[0].length||y<0) return false;
        if(vis[x][y]) return false;
        return true;
    }
}
```

