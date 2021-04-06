# Find Median from Data Stream

### 问题

一个集合，两种操作。插入操作：将某个数插入到该集合中；查询操作：返回该集合的所有的数的中位数（为偶数那么将中间两数的平均值返回）。请实现这几个功能。

### 解决方案：优先队列，查询复杂度$O(1)$，插入复杂度$O(logn)$

半数分别由两个不同的优先队列维护。每次插入，需要经过两个优先队列（即第一个队列先offer，第二个队列再offer第一个队列poll的）。这里将中位数（奇数情况）交给small队列维护。

```java
class MedianFinder {
    
    PriorityQueue<Integer> large;
    PriorityQueue<Integer> small;
    boolean even=true;

    public MedianFinder() {
        large=new PriorityQueue<>(Collections.reverseOrder());
        small=new PriorityQueue<>();
    }
    
    public void addNum(int num) {
        if(even){
            large.offer(num);
            small.offer(large.poll());
        }else{
            small.offer(num);
            large.offer(small.poll());
        }
        even=!even;
    }
    
    public double findMedian() {
        if(even) return (large.peek()+small.peek())/(double)2;
        else return small.peek();
    }
}
```

