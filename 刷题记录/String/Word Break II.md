# Word Break II

### 问题

给一个没有空格的只包含小写字母的句子，给一个字典。向该句子中插入空格，使得句子中的每一个词都能在字典中找到，求所有的合法的句子。句子长度为n，字典中词语的数量为m。

### 解决方案：字典树+dfs，时间复杂度$O(nm)$

在dfs中，字典树中的node与当前需要匹配的pos对应。node通过`s[pos]`到达下一个node。要注意这种关系。

如果当前node为空，直接返回，因为pos会在node为空之前就到达`s.length`。`s[pos]`表示这次需要匹配的字母。如果`pos==s.length&&node!=null`，那么需要判断node是否为叶子节点，是则将当前串加入结果集。

```java
class Solution {
        StringBuilder sb;
        List<String> ans;
        Node root;

        public List<String> wordBreak(String s, List<String> wordDict) {
            sb = new StringBuilder();
            ans = new ArrayList<>();
            root = new Node();
            for (String word : wordDict) {
                Node node = root;
                for (int i = 0; i < word.length(); i++) {
                    int id = word.charAt(i) - 'a';
                    if (node.next[id] == null) node.next[id] = new Node();
                    node = node.next[id];
                }
                node.flag = true;
            }
            dfs(s, 0, root);
            return ans;
        }

        class Node {
            Node[] next = new Node[26];
            boolean flag = false;
        }

        public void dfs(String s, int pos, Node node) {
            if (pos > s.length() || node == null) return;
            if (pos == s.length()) {
                if (node.flag) {
                    ans.add(sb.toString());
                }
                return;
            }
            //pos<s.length
            if (node.flag) {
                sb.append(" ");
                dfs(s, pos, root);
                sb.deleteCharAt(sb.length() - 1);
            }
            int id = s.charAt(pos) - 'a';
            sb.append(s.charAt(pos));
            dfs(s, pos + 1, node.next[id]);
            sb.deleteCharAt(sb.length() - 1);
            return;
        }
    }
```

