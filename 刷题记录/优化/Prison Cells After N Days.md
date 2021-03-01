# Prison Cells After N Days

### 问题

考虑长度为8的01数组。

定义从数组a到数组b的变化：

+ 如果`a[i-1]==a[i+1]`，那么`b[i]=1`，否则`b[i]=0`
+ `b[0]=b[7]=0`

输入一个字符串`int[] cells`以及变换次数`n`，求变换结果。

### 解决方案：时间复杂度$O(1)$

注意`n`可以取很大的值。

之前的想法是使用hashmap来记录变换的轨迹，如果遇到相同的起始值，那么直接拿来用。但是还是TLE了（主要是在`n`的循环）。

最后采用周期来直接计算，用list记录变换序列，map记录变换序号与序列的关系。如果发现重复的序列，直接计算返回结果。

```java
class Solution {
        public int[] prisonAfterNDays(int[] cells, int n) {
                int pre=getCode(cells);
                int next=0;
                List<Integer> list=new ArrayList<>();
                HashMap<Integer, Integer> mp=new HashMap<>();
                for(int i=0; i<n; i++){
                        int[] pre_cells=getCells(pre);
                        for(int j=0; j<8; j++){
                                if(j==0||j==7) next=next*10;
                                else next=next*10+1-(pre_cells[j-1]^pre_cells[j+1]);
                        }
                        if(mp.containsKey(next)){
                            	//注意，need是需要变换的次数
                                int need=n-i-1;
                                int offset=mp.get(next);
                                int cycle=list.size()-offset;
                                int ans=list.get((need%cycle)+offset);
                                return getCells(ans);
                        }
                        pre=next;
                        next=0;
                        mp.put(pre, list.size());
                        list.add(pre);
                }
                return getCells(pre);
        }
        public int getCode(int[] cells){
                int ans=0;
                for(int x:cells){
                        ans=ans*10+x;
                }
                return ans;
        }
        public int[] getCells(int code){
                int[] ans=new int[8];
                for(int i=7; i>=0; i--){
                        ans[i]=code%10;
                        code/=10;
                }
                return ans;
        }
}
```

