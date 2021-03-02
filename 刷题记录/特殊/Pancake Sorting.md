# Pancake Sorting

### 问题

有一个乱序的数组arr，长度为n，元素为不重复的1~n。现在能进行以下操作：

+ 选择一个k，1<=k<=n
+ 将arr[0]~arr[k-1]这k个数进行翻转

要求经过最多10*arr.length操作，使得数组为升序。

求任意一个合法的操作序列。

### 解决方案：时间复杂度$O(n^2)$

从最大数开始，逐一将它们回到原位。

后缀是有序的大数序列，因此只需要对前缀进行翻转。

例如现在最大数n的下标为k，那么将(0, k-1)翻转，使得最大数n在首位，之后将(0, n-1)翻转，使得n回到原位。最大的翻转次数为2*arr.length。

```java
class Solution {
        public List<Integer> pancakeSort(int[] arr) {
                List<Integer> ans=new ArrayList<>();
                for(int i=arr.length; i>0; i--){
                        int index=findIndex(arr, i);
                        if(i==index+1) continue;
                        ans.add(index+1);
                        flip(arr, 0, index);
                        ans.add(i);
                        flip(arr, 0, i-1);
                }
                return ans;
        }
        
        public int findIndex(int[] arr, int num){
                for(int i=0; i<arr.length; i++){
                        if(arr[i]==num) return i;
                }
                return -1;
        }
        
        public void flip(int[] arr, int lo, int hi){
                int tmp=0;
                while(lo<=hi){
                        tmp=arr[lo];
                        arr[lo]=arr[hi];
                        arr[hi]=tmp;
                        lo++;
                        hi--;
                }
        }
}
```

