# LIS

### 问题

给定一个长度为N的数组A，找出一个最长的单调自增子序列（不一定连续，但顺序必须为正序）

### 解法一：LCS, $o(n^2)$ 

这个问题可以转换成LCS问题。

先把A排序得到A'找出A和A'的最长公共子序列即可。

设从$c[i,j]$表示$A[1...i]$与$A'[1...j]$的LCS长度，则有：
$$
c[i,j] =
\begin{cases}
0, & if\;i=0\;or\;j=0\\
c[i-1,j-1]+1, & if\;A[i]=A'[j]\\
max\{c[i-1,j], c[i,j-1]\}, & if\;A[i]\neq A'[j]
\end{cases}
$$

### 解法二：普通DP，$O(N^2)$

设$c[i]$表示$A[1...i]$中的以$A[i]$结尾的LIS值，则有：
$$
\begin{aligned}
c[i]&=max\{c[j]+1\;|\;j<i\;and\;A[j]\leq A[i]\}\\
ans&=max\{c[i]\;|\;i=1...N\}
\end{aligned}
$$
当然还有另一种定义$c[i]$表示$A[1...i]$中LIS值，有：
$$
c[i]=max\{
\begin{aligned}
&c[j]+1,\;if\;A[j]\leq A[i]\\
&c[j],\;otherwise
\end{aligned}\}\\
ans=c[N]
$$
当数据量较大时，前者有剪枝，耗时较少。

### 解法三：贪心+二分查找，$O(NlogN)$

贪心：如果子序列长度相同，终点越小，后面的增长潜能越大

则设$c[i]$表示LIS长度为i的最小尾数，则$c[k]$为递增序列$\forall A[j]\in A$有：
$$
if\;c[i+1]>A[j]>c[i]\;then\;c[i+1]=A[j]
$$


初始化$c[1...N]=\infty$。

上式也可二分优化成$O(NlogN)$算法。