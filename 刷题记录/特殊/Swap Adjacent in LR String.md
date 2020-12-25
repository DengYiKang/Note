# Swap Adjacent in LR String

### 问题

`start`为一个字符串，只包含`L, R, X`，可以做变换`XL->LX`以及`RX->XR`，`end`为另一字符串，问`start`能否变换成`end`

### 解决方案：时间复杂度$O(n)$

`L`与`R`不能交换位置，因此`L, R`的相对位置固定，即`start` 与`end`如果去除掉`X`后，两个字符串应该相等。

对于一一对应的`L`来说，假设`start[i]`对应`end[j]`，它们的取值为`L`，那么必须有`i>=j`，对`R`来说同理。

```java
class Solution {
    public boolean canTransform(String start, String end) {
        char[] s1=start.toCharArray();
        char[] s2=end.toCharArray();
        List<int[]> list1=new ArrayList<>();
        List<int[]> list2=new ArrayList<>();
        for(int i=0; i<s1.length; i++){
            if(s1[i]=='L') list1.add(new int[]{0, i});
            else if(s1[i]=='R') list1.add(new int[]{1, i});
            if(s2[i]=='L') list2.add(new int[]{0, i});
            else if(s2[i]=='R') list2.add(new int[]{1, i});
        }
        if(list1.size()!=list2.size()) return false;
        for(int i=0; i<list1.size(); i++){
            int[] a=list1.get(i);
            int[] b=list2.get(i);
            if(a[0]!=b[0]) return false;
            if(a[0]==0&&a[1]<b[1]) return false;
            if(a[0]==1&&a[1]>b[1]) return false;
        }
        return true;
    }
} 
```

