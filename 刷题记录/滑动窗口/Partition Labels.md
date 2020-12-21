# Partition Labels

### 问题

`S`为一个只包含26个小写字母的字符串，要求对它进行分割，使得所有的小写字母出现在至多一个划分中，并且要求尽可能的划分（分割数最大）。

### 解决方案：贪心+滑动窗口，时间复杂度$O(n)$ 

维护一个区间，`left`和`right`表示当前划分的左右两端。

首先初始化`int[] first`和`int[] last`，保存26个字母的最早和最晚出现位置。

在遍历的过程中，每次将`right`更新为当前字母的最晚出现的位置。如果当前位置在区间外，那么将该区间就是一个符合题意的分割。

```java
class Solution {
    public List<Integer> partitionLabels(String S) {
        int[] first=new int[26];
        int[] last=new int[26];
        char[] str=S.toCharArray();
        Arrays.fill(first, -1);
        for(int i=0; i<str.length; i++){
            int id=str[i]-'a';
            if(first[id]==-1) first[id]=i;
            last[id]=i;
        }
        int left=0, right=0;
        List<Integer> ans=new ArrayList<>();
        for(int i=0; i<str.length; i++){
            if(i>right){
                //开始新的区间
                ans.add(right-left+1);
                left=i;
                right=last[str[i]-'a'];
            }else{
                //更新区间右端
                right=Math.max(right, last[str[i]-'a']);
            }
        }
        //注意别忘了最后一个区间
        ans.add(right-left+1);
        return ans;
    }
}
```

