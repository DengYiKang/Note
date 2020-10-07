# Top K Frequent Elements

### 问题

现有一个整型数组，给出出现次数最多k个的数。

### 解决方案：桶排序，时间复杂度$O(n)$，空间复杂度$O(n)$

key为频率，val为数即可。

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        List<Integer>[] bucket = new List[nums.length + 1];
	    Map<Integer, Integer> frequencyMap = new HashMap<Integer, Integer>();
        for (int n : nums) {
		    frequencyMap.put(n, frequencyMap.getOrDefault(n, 0) + 1);
	    }
        for (int key : frequencyMap.keySet()) {
            int frequency = frequencyMap.get(key);
            if (bucket[frequency] == null) {
                bucket[frequency] = new ArrayList<>();
            }
            bucket[frequency].add(key);
        }
        int[] res = new int[k];
        int tail=0;
        for (int pos = bucket.length - 1; pos >= 0 && tail < k; pos--) {
            if (bucket[pos] != null) {
                for(Integer i:bucket[pos]){
                   res[tail++]=i; 
                }
            }
        }
        return res; 
    }
}
```

