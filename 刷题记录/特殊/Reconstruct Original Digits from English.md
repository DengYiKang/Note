# Reconstruct Original Digits from English

### 问题

现有一个字符串，由数字`0~9`得到英文单词组成，但是乱序。输出组成该字符串的英文单词对应的数字，升序排列。

**Example 1:**

```
Input: "owoztneoer"

Output: "012"
```

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

只要找到每个单词区别于其他单词独一无二的字母即可。

```java
class Solution {
    int[] num;
    int[] cnt;
    List<String> list;
    public String originalDigits(String s) {
        num=new int[10];
        cnt=new int[26];
        list=new ArrayList<>();
        list.add("zero");
        list.add("one");
        list.add("two");
        list.add("three");
        list.add("four");
        list.add("five");
        list.add("six");
        list.add("seven");
        list.add("eight");
        list.add("nine");
        for(char ch:s.toCharArray()){
            int id=ch-'a';
            cnt[id]++;
        }
        deal(0, 'z', 1);
        deal(2, 'w', 1);
        deal(4, 'u', 1);
        deal(5, 'f', 1);
        deal(7, 'v', 1);
        deal(6, 's', 1);
        deal(1, 'o', 1);
        deal(9, 'n', 2);
        deal(8, 'i', 1);
        deal(3, 't', 1);
        StringBuilder sb=new StringBuilder();
        for(int i=0; i<10; i++){
            char ch=(char)('a'+i);
            for(int j=0; j<num[i]; j++){
                sb.append(String.valueOf(i));
            }
        }
        return sb.toString();
    }
    void deal(int pos, char ch, int coe){
        num[pos]=cnt[ch-'a']/coe;
        for(char mch:list.get(pos).toCharArray()){
            cnt[mch-'a']-=num[pos];
        }
    }
}
```

