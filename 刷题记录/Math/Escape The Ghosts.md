# Escape The Ghosts

### 问题

你从`(0,0)`这个点出发到`(target[0], target[1])`这个点进行逃脱。其他一些位置有鬼魂，每次行动时，你们都可以往任意方向走一步或者留在原地。求问是否有逃脱的可能。

### 解决方案：时间复杂度$O(n)$

要想这人无法逃脱，鬼魂先一步占住`target`就行。

```java
class Solution {
    public boolean escapeGhosts(int[][] ghosts, int[] target) {
        int need=Math.abs(target[0])+Math.abs(target[1]);
        for(int[] ghost:ghosts){
            int d=Math.abs(ghost[0]-target[0])+Math.abs(ghost[1]-target[1]);
            if(d<=need) return false;
        }
        return true;
    }
}
```

