# Largest Number

### 问题

输入一个数组A，输出一个由A的所有元素组合而成的最大的数

### 解决方案：$O(nlogn)$

```java
class Solution {
    public String largestNumber(int[] nums) {
        if(nums==null||nums.length==0) return "";
        StringBuilder ans=new StringBuilder();
        List<String> A=new ArrayList<>();
        for(int x:nums){
            A.add(String.valueOf(x));
        }
        Collections.sort(A, new Comparator<String>(){
            @Override
            public int compare(String a, String b){
                String order1=a+b;
                String order2=b+a;
                return order2.compareTo(order1);
            }
        });
        for(String s:A){
            ans.append(s);
        }
        String result=ans.toString();
        if(result.charAt(0)=='0') result="0";
        return result;
    }
}
```

错误的代码：(不知道逻辑上哪里错了...)

```java
class Solution {
    public String largestNumber(int[] nums) {
        if(nums==null||nums.length==0) return "";
        StringBuilder ans=new StringBuilder();
        List<String> A=new ArrayList<>();
        for(int x:nums){
            A.add(String.valueOf(x));
        }
        Collections.sort(A, new Comparator<String>(){
            @Override
            public int compare(String a, String b){
                int x=0;
                while(x<a.length()&&x<b.length()){
                    if(a.charAt(x)>b.charAt(x)) return -1;
                    else if(a.charAt(x)<b.charAt(x)) return 1;
                    x++;
                }
                if(x==a.length()&&x==b.length()) return 1;
                if(x==a.length()){
                    if(b.charAt(x)>a.charAt(0)) return 1;
                    else return -1;
                }else if(x==b.length()){
                    if(a.charAt(x)>b.charAt(0)) return -1;
                    else return 1;
                }
                return -1;
            }
        });
        for(String s:A){
            ans.append(s);
        }
        String result=ans.toString();
        if(result.charAt(0)=='0') result="0";
        return result;
    }
}
```

