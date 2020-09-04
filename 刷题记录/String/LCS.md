# LCS

### 问题

给定两个字符串$A[1...N_1],B[1...N_2]$，求$A,B$ 的最长公共子串的长度。

### 基本思路，$O(N^2)$

设$c[i,j]$表示$A[1...i]$与$B[1...j]$的LCS值，则有：
$$
c[i,j]=
\begin{cases}
0,\;&if\;i=0\;or\;j=0\\
c[i-1,j-1]+1,\;&if\;A[i]=B[j]\\
max\{c[i-1,j],c[i,j-1]\},\;&if\;A[i]\neq B[j]
\end{cases}
$$

