# Binary String With Substrings Representing 1 To N

### 问题

01字符串`S`，大于1的整型`N`，问`S`是否含有所有`1～N`的子串？

### 解决方案：时间复杂度$O(30n)\;n=S.len$

`N`是整型，$1->10^{9}$，范围太大，需要从字符串`S`的角度考虑。

因为$10^9$最多30 bit，因此只需要考虑串中的所有最长为30 bit的子串。维护一个set，所有能生成的数字都加入到这个set中，最后如果set.len!=N，那么返回false。

```java
class Solution {
        public boolean queryString(String S, int N) {
                int len=S.length();
                if(len*len<N) return false;
                HashSet<Integer> nums=new HashSet<>();
                for(int i=0; i<len; i++){
                        for(int j=i; j<len&&(j-i)<30; j++){
                                String str="";
                                if(i!=j) str=S.substring(i, j+1);
                                else str=String.valueOf(S.charAt(i));
                                int num=getInt(str);
                                if(num>0&&num<=N) nums.add(num);
                        }
                }
                return nums.size()==N;
        }
        int getInt(String s){
                int ans=0;
                for(int i=0; i<s.length(); i++){
                        ans<<=1;
                        ans+=s.charAt(i)-'0';
                }
                return ans;
        }
}
```



