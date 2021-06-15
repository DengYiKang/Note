# report01

## Adaptive Federated Learning in Resource Constrained Edge Computing Systems

这篇论文主要工作为：

1、理论上分析了基于同步梯度下降的联邦学习的收敛界

2、提出动态的控制算法，动态地调整global aggregation的频率，在有限的资源下将learning loss最小化

这篇论文研究的是联邦学习，需要考虑以下几点：

1、不同特征的数据集（非独立同分布， non-i.i.d）

2、两次全局aggregation之间local update的次数

上面的第一个因素将会在后续引入一个$\delta$来考虑，第二个因素就是这篇论文的主要目的。

<img src="/home/yikang/Document/gitRep/Note/pic/136.png" style="zoom:80%;" />

基本框架是分布式梯度下降：

![](/home/yikang/Document/gitRep/Note/pic/137.png)

首先所有的节点的w参数初始化为一致的w，然后各个节点内部进行local update，各个节点相互独立地进行梯度下降，当进行了多次local update后，各个节点将w参数传到服务器，服务器将所有的w参数做个平均然后分发到各个节点作为新的w来使用。

如果再引入资源的限制，如内存、时间的限制，那么各个节点所能进行的local update以及global update的次数是受到了限制的，当资源耗尽时必须终止计算，将当前结果作为最终结果并退出。

这里假定每经历k次local update进行一次global update，那么问题在于如何对k进行取值从而在有限的资源下使loss function的值降到最低？

这篇论文首先理论上分析了基于同步梯度下降的联邦学习的收敛界。

将global update的时间点作为单位，将连续的local update的时间点作为一个区间来考虑，$w$为联邦学习下的参数，$v$为集中式的学习（类似于每一次都是globa update），那么就有：

<img src="/home/yikang/Document/gitRep/Note/pic/138.png" style="zoom:80%;" />

在每个Interval区间里，两种模式下随着迭代的次数增大loss function将逐渐减小，因为集中式的学习是基于全局的数据，而联邦学习是local与global update相间的，因此集中式的性能往往比联邦学习的性能好。为了后续便于对每个区间的分析，在每个Interval的交界，将集中式的学习得参数调整到与联邦学习的参数相同。通过$F(w)$与$F(v)$的差值来分析联邦学习的收敛界。

优化的目标函数以及约束如下：

![](/home/yikang/Document/gitRep/Note/pic/145.png)

首先有以下假设：

![](/home/yikang/Document/gitRep/Note/pic/139.png)

![](/home/yikang/Document/gitRep/Note/pic/140.png)

假设loss function是凸函数、Lipschitz以及smooth。

推出：

![](/home/yikang/Document/gitRep/Note/pic/141.png)

然后定义$\delta$值，来衡量各个数据集的差异性（即考虑non-i.i.d的问题，非独立同分布）

![](/home/yikang/Document/gitRep/Note/pic/142.png)

然后推出以下结论：

<img src="/home/yikang/Document/gitRep/Note/pic/143.png" style="zoom:80%;" />

当τ>1时，$x=t-(k-1)$会特别大，那么$h(x)$也会特别大，那么联邦学习的准确率会远远比集中式的要低。注意到$h(x)$正比于$\delta$，这很好理解，如果局部梯度与全局梯度相差太大，对于该局部来说用全局梯度计算出的准确率会更低。

之后推出：

![](/home/yikang/Document/gitRep/Note/pic/146.png)

![](/home/yikang/Document/gitRep/Note/pic/144.png)

$F(w^*)$是最优的，可以看做一个常数，所以对$F(w^f)$的优化可以看做是对$F(w^f)-F(w^*)$的优化，那么目标函数就变成：

![](/home/yikang/Document/gitRep/Note/pic/147.png)

τ没有给出计算方法，是通过搜索来找到的，这篇论文给出了τ的一个上界：

<img src="/home/yikang/Document/gitRep/Note/pic/148.png" style="zoom:80%;" />

可以在$[1,τ_{max}]$之间线性搜索$τ$的值。

最终算法如下：

aggregator：

![](/home/yikang/Document/gitRep/Note/pic/149.png)

![](/home/yikang/Document/gitRep/Note/pic/150.png)

edge node：

![](/home/yikang/Document/gitRep/Note/pic/151.png)

结果：

![](/home/yikang/Document/gitRep/Note/pic/153.png)

![](/home/yikang/Document/gitRep/Note/pic/154.png)



![](/home/yikang/Document/gitRep/Note/pic/152.png)

### 总结

理论分析了在有限的资源下如何安排local update和global update的次数来使得loss最小化。

因为一开始假设的loss function是凸函数，但是对于深度神经网络不适用，并且实验大部分都是使用的传统的机器学习算法进行比较，如逻辑回归、SVM等，这些算法的激活函数都是凸函数。

## Federated Machine Learning: Concept and Applications

这是一篇关于FL的一篇综述。介绍了FL的概念、FL的分类等。

### FL的定义

定义N个数据所有者${F_1, ...,F_N}$，这些所有者希望在不泄露自己的数据集的情况下同其他所有者共同训练，设这种训练方式的精度为$V_{FED}$。传统的方法是将所有的数据集合并起来作为一个大的数据集来训练，设这种方式的精度为$V_{SUM}$。$\delta=|V_{FED}-V_{SUM}|$，那么称FL算法有$\delta$-accuracy loss。

### FL中的不同的隐私技术

#### Secure Multi-party Computation (SMC)

安全多方计算。

理想情况下，用户除了其输入和输出什么也不知道。在某些情况下，如果提供了安全保证，那么部分知识的披露被认为是可以接受的。

#### Differential Privacy

差分隐私。

在数据中添加噪声，或使用泛化方法隐藏某些敏感属性，直到第三方无法区分个人，从而使数据无法恢复以保护用户隐私。

#### Homomorphic Encryption

同态加密。

通过加密机制下的参数交换来保护用户数据隐私。

### FL的分类

根据数据的不同分布，FL可以分为水平FL、垂直FL、迁移学习FL。

假设$X$表示特征空间，$Y$表示标签空间，$I$表示ID空间，那么数据集$D$可以由$(I, X, Y)$来表示。

#### 水平FL

数据集共享相同的特征空间但样本不同的场景。例如两家不同的银行，他们的业务非常类似，特征空间是相同的，但是由于地区的原因，用户组的交集非常小。

此时有：
$$
X_i=X_j,\;Y_i=Y_j,\;I_i\neq I_j,\;\forall D_i,D_j,\;i\neq j
$$
<img src="/home/yikang/Document/gitRep/Note/pic/158.png" style="zoom: 80%;" />

#### 垂直FL

两个数据集共享相同的样本ID空间但在特征空间中不同的情况。例如同一区域的一家银行一家电子商务公司，他们的用户交集将会非常大，但是由于业务的不同，他们的特征空间将不同。

此时有：
$$
X_i\neq X_j,\;Y_i\neq Y_j,\;I_i=I_j,\;\forall D_i,D_j,\;i\neq j
$$

<img src="/home/yikang/Document/gitRep/Note/pic/159.png" style="zoom:80%;" />

#### 迁移学习FL

两个数据集不仅样本不同而且特征空间也不同的场景。

此时有：
$$
X_i\neq X_j,\;Y_i\neq Y_j,\;I_i\neq I_j,\;\forall D_i,D_j,\;i\neq j
$$

<img src="/home/yikang/Document/gitRep/Note/pic/160.png" style="zoom:80%;" />

### FL系统的体系结构

#### 水平FL

一个典型的假设是参与者是诚实的，而服务器是诚实但好奇的，因此不允许任何参与者向服务器泄露信息。

<img src="/home/yikang/Document/gitRep/Note/pic/161.png" style="zoom:80%;" />

#### 垂直FL

为了确保训练过程中数据的机密性，第三方合作者C参与其中。一个典型的假设是合作者C是诚实的，不与A或B串通，但甲方和乙方是诚实的，但彼此好奇。

<img src="/home/yikang/Document/gitRep/Note/pic/162.png" style="zoom:80%;" />

#### 迁移学习FL

与垂直FL的体系结构相同，但是一些细节不同。

### 其他

#### Federated Learning vs Distributed Machine Learning

横向的联邦学习与分布式机器学习有些相似。

相比分布式机器学习，横向的联邦学习对本地数据具有完全的自治，可以决定何时、如何加入联邦学习，并且也强调隐私保护。

#### 应用

以智能零售为例。其目的是利用机器学习技术为客户提供个性化服务，主要包括产品推荐和销售服务。智能零售业务涉及的数据特征主要包括用户购买力、用户个人偏好和产品特征。在实际应用中，这三种数据特征很可能分散在三个不同的部门或企业中。例如，一个用户的购买力可以从她的银行储蓄中推断出来，她的个人偏好可以从她的社交网络中分析出来，而产品的特性则由电子商店记录下来。在这种情况下，我们面临两个问题。首先，为了保护数据隐私和数据安全，银行、社交网站和电子购物网站之间的数据壁垒很难打破。因此，不能直接聚合数据来训练模型。第二，三方存储的数据通常是异构的，传统的机器学习模型不能直接处理异构数据。那么这个时候就要引入联邦学习来处理了。

## Federated Meta-Learning with Fast Convergence and Efficient Communication

federated learning具有两个挑战：

+ 统计挑战：分散的数据是非IID，高度个性化和异构的，这会导致模型精度显著降低
+ 系统挑战：设备的数量通常比传统分布式设置中的设备数量大一个数量级。此外，每个设备在存储、计算和通信容量方面可能具有显著的限制。

> 至于为什么能缓解非IID的问题，我是这样思考的：首先FedAvg本身就能有效缓解非IID的问题，而FedMeta类似于精简的FedAvg（如果不考虑support set和query set的话），那么它肯定能多少缓解非IID的问题；且考虑到support set和query set，与MAML的思想（注重$\phi$的潜力而非当前）类似，因此更（？）能缓解非IID的问题，同时采样减少了计算量。

这篇文章将meta learning与federated learning结合起来，用来解决以上两个问题。

<img src="/home/yikang/Document/gitRep/Note/pic/172.png" style="zoom:80%;" />

可以看出，先在client中采样，然后将每个client视为一个task，进行meta learning。

## FedFast: Going beyond Average for Faster Training of Federated Recommender Systems

### 摘要

提出了一个基于FL的RS算法，在避免暴露用户的隐私的情况下进行推荐，同时利用user之间的相似度来加速收敛。

### 模型

<img src="/home/yikang/Document/gitRep/Note/pic/174.png" style="zoom:80%;" />

![](/home/yikang/Document/gitRep/Note/pic/175.png)

算法流程：

<img src="/home/yikang/Document/gitRep/Note/pic/176.png" style="zoom:80%;" />

这是很标准的FL框架，ClientUpdate()过程即local update。先进行采样，后进行多轮local update后进行一次aggragate。

<img src="/home/yikang/Document/gitRep/Note/pic/177.png" style="zoom:80%;" />

重点是ActvAGG：

这里先引入delegate与candidate两个概念，delegate是指被sample选中的client，而candidate是未被选中的client。

<img src="/home/yikang/Document/gitRep/Note/pic/178.png" style="zoom:80%;" />

对于non-embedding components，直接加权平均。

对于item embeddings，各个delegate的contribution正比于w的变化（如果某个delegate变化越剧烈，那么全局收到的影响也越大，突出个性化？）

<img src="/home/yikang/Document/gitRep/Note/pic/179.png" style="zoom:80%;" />

对于user embeddings，$w[U_k]\gets w^k[U_k]$，这里直接赋值应该是假设各个client的user不重复（item是假定有重复的）。

赋值完之后，更新cluster，保证cluster内部是最新的相似信息。

每一个cluster如果存在delegate，那么这个cluster的所有candidate获得这些delegate的update的均值用于自身的update。这种updating sharing的方式能够加速收敛。在前期这种方式会很有效，但是到后期就不行了，因此在后期为了避免不必要的计算，采用递减的$\gamma$来控制。

### 创新点

+ 将FL与RS系统相结合
+ 各个delegate的contribution正比于w的变化，如果某个delegate变化越剧烈，那么全局受到的影响也越大，突出个性化？
+ 利用cluster内的相似的user共享updating，加速收敛

### 缺陷

这篇文章需要实现了解client之间的相似度，但是如何实现这篇文章并没有提及。

## Personalized Cross-Silo Federated Learning on Non-IID Data(FedAMP)

### 摘要

提出了一种新的方法FedAMP，它利用联邦参与式消息传递来促进相似客户的协作，建立了凸模型和非凸模型的FedAMP的收敛性，并提出了一种启发式方法来进一步提高FedAMP在客户采用深度神经网络作为个性化模型时的性能。

一些联合学习方法试图通过在训练全局模型后执行额外的微调步骤来解决问题，虽然这些方法在某些情况下有效，但它们不能系统地解决问题，作者认为这些方法的根本错误在于使用全局的模型来对所有client进行训练。

因此，这篇文章考虑通过迭代地鼓励相似的客户端进行更多的协作，自适应地促进客户端之间潜在的成对协作，从而解决了具有挑战性的个性化cross-silo学习问题。

### 问题建立

目标函数：
$$
min_W\{\mathcal{G}(W):=\sum_{i=1}^m F_i(w_i)+\lambda\sum_{i<j}^m A(||w_i-w_j||^2)\}
$$
其中，$F_i$为local training objective function，$W=[w_1,...,w_m]$，$\lambda>0$为正则化参数。

$\sum_{i=1}^m F_i(w_i)$是所有client的trainning loss的和，$\lambda\sum_{i<j}^m A(||w_i-w_j||^2)$为attention-inducing函数项来引导client间相互合作。

$A$是一个非线性的函数，满足：

+ $A$在$[0,\infty)$上递增，凹函数，$A(0)=0$
+ $A$连续可微
+ 0+可导

这篇文章中$A$的选择：$A(||w_i-w_j||^2)=1-e^{-||w_i-w_j||^2/\sigma}$

#### General Method

令$\mathcal{F}(W):=\sum_{i=1}^m F_i(w_i)$，$\mathcal{A}(W):=\sum_{i<j}^m A(||w_i-w_j||^2)$，那么目标函数可以重写成：
$$
min_W\{\mathcal{G}(W):=\mathcal{F}(W)+\lambda\mathcal{A(W)}\}
$$
使用iteratively optimize（交替优化）的方法来求解：
$$
U^k=W^{k-1}-\alpha_k\nabla\mathcal{A}(W^{k-1})\\
W^k=argmin_W\mathcal{F}(W)+\frac{\lambda}{2\alpha_k}||W-U^k||^2
$$

#### FedAMP

令$U^k=[u_1^k,...,u_m^k]$，那么有：
$$
u_i^k=\lgroup1-\alpha_k\sum_{j\neq i}^m A'(||w_i^{k-1}-w_j^{k-1}||^2)\rgroup\cdot w_i^{k-1}\\+\alpha_k\sum_{j\neq i}^mA'(||w_i^{k-1}-w_j^{k-1}||^2)\cdot w_j^{k-1}\\
=\xi_{i,1}w_1^{k-1}+...+\xi_{i,m}w_m^{k-1}
$$

$$
where\;\xi_{i,j}=\alpha_kA'(||w_i^{k-1}-w_j^{k-1}||^2),(i\neq j)
$$

其中$\xi_{i,1}+...,\xi_{i,m}=1$。

对client i训练时，j与i越相似，那么$\xi_{i,j}$就越大，即占比越高。这里可以看出FedAMP能够通过正反馈环迭代的促使相似模型参数有着更强的合作，自适应地、隐式地把相似client聚在一起。

各个client将自己的$w_i^{k-1}$上传到服务器，服务器计算出$u_i^k$后分发给client用来计算$w_i^k$：
$$
w_i^k=argmin_{W\in\mathbb{R}^d}F_i(w)+\frac{\lambda}{2\alpha_k}||w-u_i^k||^2
$$
算法流程如下：

![](/home/yikang/Document/gitRep/Note/pic/173.png)

#### HeurFedAMP

Heuristic Improvement of FedAMP on Deep Neural Networks.

深度网络中计算欧氏距离$|w_i-w_j|$维数过高，因此提出一个更好的方法代替$A'(||w_i-w_j||^2)$:
$$
\xi_{i,j}=\frac{e^{\sigma cos(w_i^{k-1},w_j^{k-1})}}{\sum_{h\neq i}^m e^{\sigma cos(w_i^{k-1}, w_h^{k-1})}}\cdot(1-\xi_{i,i})
$$

#### 源代码

[https://t.ly/nGN9]