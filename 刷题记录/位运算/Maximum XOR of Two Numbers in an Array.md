# Maximum XOR of Two Numbers in an Array

### 问题

现有一个整型数组`nums`，求得`nums[i] xor nums[j]`的最大值，其中`0<=i<=j<=n`

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

基本思想是优先求得结果的前缀的最大值。

以前缀的长度作为阶段的划分，对于当前的前缀长度4，例如已有结果`1100`，现搜寻长度为5时的结果。我们期望有结果`11001`（不考虑`11000`是因为该bit位上有且只有两种结果，若`11001`不存在，那么直接左移即可）。注意有公式`a^b=c`等价于`a^c=b`，其中`a,b`均为`nums`中的长度为5的前缀，我们现已得`c`，扫描`nums`中的长度为5的前缀`a`，那么可以计算出`b`，根据`b`是否在`nums`中长度为5的前缀的集合中，若存在那么结果更新为`11001`，否则直接左移成`11000`。

```java
public int findMaximumXOR(int[] nums) {
        int maxResult = 0; 
        int mask = 0;
        /*The maxResult is a record of the largest XOR we got so far. if it's 11100 at i = 2, it means before we reach the last two bits, 11100 is the biggest XOR we have, and we're going to explore whether we can get another two '1's and put them into maxResult
        This is a greedy part, since we're looking for the largest XOR, we start 
        from the very begining, aka, the 31st postition of bits. */
        for (int i = 31; i >= 0; i--) {
            
            //The mask will grow like  100..000 , 110..000, 111..000,  then 1111...111
            //for each iteration, we only care about the left parts
            mask = mask | (1 << i);
            
            Set<Integer> set = new HashSet<>();
            for (int num : nums) {
                
				/*we only care about the left parts, for example, if i = 2, then we have
                {1100, 1000, 0100, 0000} from {1110, 1011, 0111, 0010}*/
                int leftPartOfNum = num & mask;
                set.add(leftPartOfNum);
            }
            
            // if i = 1 and before this iteration, the maxResult we have now is 1100, 
            // my wish is the maxResult will grow to 1110, so I will try to find a candidate
            // which can give me the greedyTry;
            int greedyTry = maxResult | (1 << i);
            
            for (int leftPartOfNum : set) {
                //This is the most tricky part, coming from a fact that if a ^ b = c, then a ^ c = b;
                // now we have the 'c', which is greedyTry, and we have the 'a', which is leftPartOfNum
                // If we hope the formula a ^ b = c to be valid, then we need the b, 
                // and to get b, we need a ^ c, if a ^ c exisited in our set, then we're good to go
                int anotherNum = leftPartOfNum ^ greedyTry;
                if (set.contains(anotherNum)) {
                    maxResult= greedyTry;
                    break;
                }
            }
            // If unfortunately, we didn't get the greedyTry, we still have our max, 
            // So after this iteration, the max will stay at 1100.
        }
        
        return maxResult;
    }
```

