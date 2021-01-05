# Friends Of Appropriate Ages

### 问题

整型数组`ages`，`ages[i]`表示第`i`位人的年龄，`A`向`B`发出邀请需要满足以下条件：

```
0.5*ages[A]+7<ages[B]<=ages[A]
ages[B]<=100||ages[A]>=100
```

- `1 <= ages.length <= 20000`.
- `1 <= ages[i] <= 120`.

问最多有多少个邀请。

### 解决方案：

注意`ages.length`特别大，但`ages[i]`特别小，因此会有许多重复的数，可以考虑用`cnt[1...120]`来存对应的数量。

```java
class Solution {
    public int numFriendRequests(int[] ages) {
        int[] cnt=new int[121];
        int res=0;
        for(int i=0; i<ages.length; i++){
            cnt[ages[i]]++;
        }
        for(int i=15; i<cnt.length; i++){
            if(cnt[i]==0) continue;
            int lo=i/2+7;
            if(lo==i/2+7) lo++;
            int tmp_sum=0;
            for(int j=lo; j<i; j++){
                tmp_sum+=cnt[j];
            }
            res+=tmp_sum*cnt[i]+cnt[i]*(cnt[i]-1);
        }
        return res;
    }
}
```

