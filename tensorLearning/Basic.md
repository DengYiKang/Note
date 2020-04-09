# Basic

[TOC]

## Fibers

A fiber of a tensor A is a vector obtained by fixing all but one A’s indices. For example, if $A = A(1:3, 1:5, 1:4, 1:7)$, then
$$
A(2,:,4,6)=A(2,1:5,4,6)=\left[
\begin{matrix}
A(2, 1, 4, 6)\\
A(2, 2, 4, 6)\\
A(2, 3, 4, 6)\\
A(2, 4, 4, 6)\\
A(2, 5, 4, 6)\\
\end{matrix}
\right]
$$
is a fiber. We adopt that convention that a fiber is a column-vector.

## Slices

A slice of a tensor A is a matrix obtained by fixing all but two of
A’s indices. For example, if $A = A(1:3, 1:5, 1:4, 1:7)$, then
$$
A(:,3,:,6)=\left[\begin{matrix}
A(1,3,1,6)&A(1,3,2,6)&A(1,3,3,6)&A(1,3,4,6)\\
A(2,3,1,6)&A(2,3,2,6)&A(2,3,3,6)&A(2,3,4,6)\\
A(3,3,1,6)&A(3,3,2,6)&A(3,3,3,6)&A(3,3,4,6)
\end{matrix}\right]
$$
is a slice. We adopt the convention that the first unfixed index in the tensor is the row index of the slice and the second unfixed index in the tensor is the column index of the slice.

## Mode-k Unfolding

In a mode-k unfolding, the mode-k fibers are assembled to produce
an $n_k-by-(N/n_k )$  matrix where $N = n_1 · · · n_d$ .

+ A Mode-1 Unfolding of $A\in{\R}^{4\times3\times2}$ 
  $$
  \left[\begin{matrix}
  a_{111}&a_{121}&a_{131}&a_{112}&a_{122}&a_{132}\\
  a_{211}&a_{221}&a_{231}&a_{212}&a_{222}&a_{232}\\
  a_{311}&a_{321}&a_{331}&a_{312}&a_{322}&a_{332}\\
  a_{411}&a_{421}&a_{431}&a_{412}&a_{422}&a_{432}
  \end{matrix}\right]
  $$

+ A Mode-2 Unfolding of $A\in{\R}^{4\times3\times2}$
  $$
  \left[\begin{matrix}
  a_{111}&a_{211}&a_{311}&a_{411}&a_{112}&a_{212}&a_{312}&a_{412}\\
  a_{121}&a_{221}&a_{321}&a_{421}&a_{122}&a_{222}&a_{322}&a_{422}\\
  a_{131}&a_{231}&a_{331}&a_{431}&a_{132}&a_{232}&a_{332}&a_{432}\\
  \end{matrix}\right]
  $$

+ A Mode-3 Unfolding of $A\in{\R}^{4\times3\times2}$ 
  $$
  \left[\begin{matrix}
  a_{111}&a_{211}&a_{311}&a_{411}&a_{121}&a_{221}&a_{321}&a_{421}&a_{131}&a_{231}&a_{331}&a_{431}\\
  a_{112}&a_{212}&a_{312}&a_{412}&a_{122}&a_{222}&a_{322}&a_{422}&a_{132}&a_{232}&a_{332}&a_{432}\\
  \end{matrix}\right]
  $$

## Kronecker Product

$B\otimes C$ is a block matrix whose $ij-th$ block is $b_{ij}C$.
$$
\left[\begin{matrix}
b_{11}&b_{12}\\
b_{21}&b_{22}\\
\end{matrix}\right]\otimes C=
\left[\begin{matrix}
b_{11}C&b_{12}C\\
b_{21}C&b_{22}C\\
\end{matrix}\right]
$$
If $A=B\otimes C$, where $B\in{\R}^{m\times n}$, $C\in{\R}^{p\times q}$, then $A\in{\R}^{p\times q\times m\times n}$. And
$$
A(i_1,i_2,i_3,i_4)=B(i_3,i_4)C(i_1,i_2)
$$

#### Some Basic Facts

$$
\begin{align}
(B\otimes C)^T\;&=\;B^T\otimes C^T\\
(B\otimes C)^{-1}\;&=\;B^{-1}\otimes C^{-1}\\
(B\otimes C)(D\otimes F)\;&=\;BD\otimes CF\\
B\otimes(C\otimes D)\;&=\;(B\otimes C)\otimes D
\end{align}
$$

Note that $B\otimes C\neq C\otimes B$ .

#### A Basic Reshaping Result

$$
Y=CXB^T\Leftrightarrow vec(Y)=(B\otimes C)vec(X)
$$

#### About the Factors

$$
\begin{align}
&LU:\;\;(P_B\otimes P_C)(B\otimes C)\;=\;(L_B\otimes L_C)(U_B\otimes U_C)\\
&Cholesky:\;\;B\otimes C\;=\;(L_B\otimes L_C)(L_B\otimes L_C)^T\\
&Schur:\;\;(Q_B\otimes Q_C)^T(B\otimes C)(Q_B\otimes Q_C)\;=\;T_B\otimes T_C\\
&QR:\;\;B\otimes C\;=\;(Q_B\otimes Q_C)(R_B\otimes R_C)\\
&SVD:\;\;B\otimes C\;=\;(U_B\otimes U_C)(\Sigma_B\otimes \Sigma_C)(V_B\otimes V_C)^T
\end{align}
$$

## The Mode-k Matrix Product

If $A\in {\R}^{n_1\times ...\times n_d}$ and $M\in {\R}^{m_k\times n_k}$, then
$$
B(\alpha_1,...,\alpha_{k-1},i,\alpha_{k+1},...\alpha_d)
\\=\\
\Sigma_{j=1}^{n_k}M(i,j)\cdot A(\alpha_1,...,\alpha_{k-1},j,\alpha_{k+1},...,\alpha_d)\\
$$
is the mode-k matrix product of M and A.

the mode-k matrix product of M and A is denoted by
$$
B=A\times_k M
$$
Thus, if $B=A\times_k M$, then $B$ is defined by $B_{(k)}=M\cdot A_{(k)}$.

#### Successive Products in the Same Mode

If $A\in{\R}^{n_1\times ...\times n_d}$ and $M_1,M_2\in{\R}^{m_k\times n_k}$, then
$$
(A\times_k M_1)\times_k M_2=A\times_k(M_1M_2)
$$

#### Successive Products in Different Modes

If $A\in{\R}^{n_1\times... \times n_d},M_k\in{\R}^{m_j\times n_j}$, and $k\neq j$ then
$$
(A\times_k M_k)\times_j M_j=(A\times_j M_j)\times_k M_k
$$

## The Tucker Product

Given $\chi\in{\R}^{n_1\times ...\times n_d}$ , find matrices $U_k\in{\R}^{n_k\times r_k}\; for\;k=1:d$, and tensor $S\in{\R}^{r_1\times ...\times r_d}$ such that
$$
\begin{align}
\chi=&S\times_1 U_1\times_2 U_2...\times_d U_d\\
=&[[S;U_1,...,U_d]]\\
=&\Sigma_{j_1=1}^{r_1}...\Sigma_{j_d=1}^{r_d}S(j_1,...j_d)\cdot U_1(:,j_1)\circ...\circ U_d(:,j_d)\\
\end{align}
$$
is a Tucker product representation of $A$.

#### In Scalar

$$
\chi(i_1,...,i_d)=\Sigma_{j_1=1}^{r_1}...\Sigma_{j_d=1}^{r_d}S(j_1,...j_d)\cdot U_1(i_1,j_1)\cdot...\cdot U_d(i_d,j_d)\\
$$


#### Multilinear Sum Formulation

Suppose $A\in{\R}^{n_1\times...\times n_d}$ and $U_k\in{\R}^{n_k\times r_k}$. If $A=S\times_1 U_1\times_2 U_2...\times_d U_d$, where $S\in{\R}^{r_1\times...\times r_d}$, then
$$
A(\textbf{i})=\Sigma_{\textbf{j}=1}^{\textbf{r}}S(\textbf{j})U_1(i_1,j_1)...U_d(i_d,j_d)
$$

#### Matrix-Vector Product Formulation

Suppose $A\in{\R}^{n_1\times...\times n_d}$ and $U_k\in{\R}^{n_k\times r_k}$. If $A=S\times_1 U_1\times_2  U_2...\times_d U_d$ where $S\in{\R}^{r_1\times...\times r_d}$, then
$$
vec(A)=(U_d\otimes...\otimes U_1)\cdot vec(S)
$$

#### Existence Situation

If $A\in{\R}^{n_1\times...\times n_d}$ and $U_k\in{\R}^{n_k\times n_k}$ is nonsingular for $k=1:d$, then
$$
\begin{align}
A&=S\times_1 U_1\times_2 U_2...\times_d U_d\\
with\;\;\;\;\;\;\;\;\;
S&=A\times_1 U_1^{-1}\times_2 U_2^{-1}...\times_d U_d^{-1}
\end{align}
$$
We will refer to the $U_k$ as the inverse factors and $S$ as the core tensor.

#### Core Tensor Unfoldings

Suppose $A\in{\R}^{n_1\times...\times n_d}$ and $U_k\in{\R}^{n_k\times n_k}$ is nonsingular for $k=1:d$. If $A=S\times_1 U_1\times_2 U_2...\times_d U_d$, then the mode-k unfolding of $S$ satisfy
$$
S_{(k)}=U_k^{-1}A_{(k)}(U_d\otimes...\otimes U_{k+1}\otimes U_{k-1}\otimes...\otimes U_1)
$$

#### HOSVD and The Truncated HOSVD

Suppose $A\in{\R}^{n_1\times...\times n_d}$ and that its mode-k unfolding has SVD 
$$
A_{(k)}=U_k \Sigma_k V_k^T
$$
for $k=1:d$. Its higher-order SVD is given by
$$
\begin{align}
A=&[[S;U_1,...U_d]]\\
=&S\times_1 U_1\times_2 U_2...\times_d U_d
\end{align}
$$
where
$$
S=A\times_1 U_1^T\times_2 U_2^T...\times_d U_d^T
$$
And then
$$
A_r=[[S(1:r_1,...,1:r_d);U_1(:,1:r_1),...U_d(:,1:r_d)]]\\
where\;r_i\leq n_i
$$
is a truncated HOSVD of $A$.

#### The Modal Unfoldings of the Core Tensor S

If $A=S\times_1 U_1\times_2 U_2...\times_d U_d$ is the HOSVD of $A\in{\R}^{n_1\times...\times n_d}$, then for $k=1:d$
$$
S_{(k)}=U_k^TA_{(k)}(U_d\otimes...\otimes U_{k+1}\otimes U_{k-1} \otimes...\otimes U_1)
$$

#### The Core Tensor S Encodes Singular Value Information

Since $U_k^TA_{(k)}V_k=\Sigma_k$ is the SVD of $A_{(k)},$ we have
$$
S_{(k)}=\Sigma_kV_k^T(U_d\otimes...\otimes U_{k+1}\otimes U_{k-1}\otimes...\otimes U_1)
$$
It follows that the rows of $S_{(k)}$ are mutually orthogonal and that the singular values of $A_{(k)}$ are the 2-norms of these rows.

## The Tensor Outer Product Operation

Applied to Order-1 Tensors
$$
\begin{align}
&A=b\circ c\;\;\;\;\;b\in{\R}^{n_1},c\in{\R}^{n_2}\\
means\;\;\;\;\;&A(i_1,i_2)=b_(i_1)c(i_2)
\end{align}
$$
Applied to Order-2 Tensors
$$
\begin{align}
&A=B\circ C\;\;\;\;\;B\in{\R}^{n_1\times r_1},C\in{\R}^{n_2\times r_2}\\
means\;\;\;\;\;&A(i_1,i_2,i_3,i_4)=B(i_1,i_2)C(i_3,i_4)
\end{align}
$$
......

#### The HOSVD as a Sum of Rank-1 Tensors

If $A=S\times_1 U_1\times_2 U_2...\times_d U_d$ is the HOSVD of $A\in{\R}^{n_1\times...\times n_d}$, then
$$
\begin{align}
&A(\textbf{i})=\Sigma_{\textbf{j}=1}^{\textbf{n}}S(\textbf{j})U_1(i_1,j_1)...U_d(i_d,j_d)\\
implies\;\;&A=\Sigma_{\textbf{j}=1}^{\textbf{n}}S(\textbf{j})\cdot U_1(:,j_1)\circ...\circ U_d(:,j_d)
\end{align}
$$

## Rank-1 Tensors (Order-3)

If $f\in{\R}^{n_1},g\in{\R}^{n_2}$, and $h\in{\R}^{n_3}$, then
$$
\begin{align}
&B=f\circ g\circ h\\
is\;defined\;by\;\;&B(i_1,i_2,i_3)=f(i_1)g(i_2)h(i_3).
\end{align}
$$
The tensor $B\in{\R}^{n_1\times n_2\times n_3}$ is a rank-1 tensor.

#### The Kronecker Product Connection

$$
B=\left[\begin{matrix}f_1\\f_2\end{matrix}\right]\circ \left[\begin{matrix}g_1\\g_2\\g_3\end{matrix}\right]\circ
\left[\begin{matrix}h_1\\h_2\end{matrix}\right]
\Leftrightarrow h\otimes g\otimes f
$$

#### The Modal Unfoldings

$$
\begin{align}
If\;\;&\\
&B=f\circ g\circ h=
\left[\begin{matrix}f_1\\f_2\end{matrix}\right]\circ
\left[\begin{matrix}g_1\\g_2\\g_3\end{matrix}\right]\circ
\left[\begin{matrix}h_1\\h_2\end{matrix}\right]\\
then\;\;&\\
&B_{(1)}=f\otimes(h\otimes g)^T\\
&B_{(2)}=g\otimes(h\otimes f)^T\\
&B_{(3)}=h\otimes(g\otimes f)^T\\
\end{align}
$$

## The CP Representation 

#### The Khatri-Rao Product

If
$$
B=\left[b_1|...|b_r\right]\in{\R}^{n_1\times r}\\C=\left[c_1|...|c_r\right]\in{\R}^{n_2\times r}
$$
then the Khatri-Rao product of $B$ and $C$ is given by
$$
B\odot C=[b_1\odot c_1|...|b_r\odot c_r]
$$
Note that Khatri-Rao products can be sequenced:
$$
F\odot G\odot H=(F\odot G)\odot H=F\odot(G\odot H).
$$


#### Kruskal Form

We say that $\chi\in{\R}^{n_1\times...\times n_d}$ is in Kruskal form if
$$
\begin{align}
\chi&=[[\lambda;U_1,...,U_d]]\\
\;&=\Sigma_{j=1}^r\lambda_j\cdot U_1(:,j)\circ...\circ U_d(:,j)\\
\end{align}\\
where\;\lambda\in{\R}^r,U_k\in{\R}^{n_k\times r}\;for\;k=1:d.
$$

$$
\chi(i_1,...,i_d)=\Sigma_{j=1}^{r}\lambda_j\cdot U_1(i_1,j)\cdot ...\cdot U_d(i_d,j)\\
vec(\chi)=\Sigma_{j=1}^{r}\lambda_j\cdot U_d(:,j)\otimes ...\otimes U_1(:,j)\\
\chi_{(k)}=U_k\cdot diag(\lambda_i)\cdot(U_d\odot...\odot U_{k+1}\odot U_{k-1}\odot...\odot U_1)^T
$$

