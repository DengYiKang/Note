# Short Encoding of Words

### 问题

A **valid encoding** of an array of `words` is any reference string `s` and array of indices `indices` such that:

- `words.length == indices.length`
- The reference string `s` ends with the `'#'` character.
- For each index `indices[i]`, the **substring** of `s` starting from `indices[i]` and up to (but not including) the next `'#'` character is equal to `words[i]`.

Given an array of `words`, return *the **length of the shortest reference string*** `s` *possible of any **valid encoding** of* `words`*.*

**Example 1:**

```
Input: words = ["time", "me", "bell"]
Output: 10
Explanation: A valid encoding would be s = "time#bell#" and indices = [0, 2, 5].
```

### 解决方案：前缀树，时间复杂度$O(n)$

将word翻转，建立前缀树。用dfs遍历树，同时记录深度`depth`，当为叶子结点时就累加`depth+1`，注意`1`是指的`#`。

```java
class Solution {
    class Node{
        Node[] next=new Node[26];
    }
    private int ans=0;
    public int minimumLengthEncoding(String[] words) {
        Node root=new Node();
        for(String word:words){
            char[] chs=word.toCharArray();
            Node cur=root;
            for(int i=chs.length-1; i>=0; i--){
                int id=chs[i]-'a';
                if(cur.next[id]==null) cur.next[id]=new Node();
                cur=cur.next[id];
            }
        }
        ans=0;
        dfs(root, 0);
        return ans;
    }
    public void dfs(Node node, int depth){
        if(node==null) return;
        boolean isLeaf=true;
        for(int i=0; i<26; i++){
            dfs(node.next[i], depth+1);
            if(node.next[i]!=null) isLeaf=false;
        }
        if(isLeaf) {
            ans+=depth+1;
        }
    }
}
```

