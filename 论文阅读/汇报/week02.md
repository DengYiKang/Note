# 第二周汇报

## Q-Learning

Critic是指如何评判一个状态到最后总共能得到多少分数。例如下面的两张图，第一张图的分数肯定很高，因为它的保护罩还完好无损，且有足够多的外星人给自己消灭，第二张图的分数就比第一张图低，它的保护罩已经没有了，且剩下的可以消灭的外星人非常少，因此这个状态的得分很低。那么我们需要learn一个$V^\pi(s)$这样一个function，输入状态s，输出得分。

<img src="../../pic/232.png" style="zoom:80%;" />

那么如何estimate $V^\pi(s)$呢？有两种方法：

第一种是蒙特卡洛式，critic观察$\pi$玩游戏，对于某个s，由$\pi$去玩，最终得到的分数$G$，然后critic将这种过程当成一个回归问题：

<img src="../../pic/233.png" style="zoom:80%;" />

这种方式的缺点就是，需要玩完一整场游戏才能得到最终的分数，不现实。

第二种是Temporal-difference(TD)的方式，对于一个片段（episode） $s_t,a_t,r_t,s_{t+1}$，$s$表示state，$a$表示action，$r$表示reward。

那么有等式成立：$V^\pi(s_t)=V^\pi(s_{t+1})+r_t$。那么利用这个等式，可以对同一个网络$ V^\pi$的不同输入得到的输出做限制，即$V^\pi(s_t)-V^\pi(s_{t+1})=r_t$，那么得到了以下的网络结构：

<img src="../../pic/234.png" style="zoom:80%;" />

对于MC方式，方差可能会特别大，当游戏流程特别长时，最终分数由之间的所有step影响，因此累积起来的变化可能会特别大。

对于TD方式，方差会特别小（因为比较的是相邻的状态），但是只考虑了某个片段，没有考虑整体，因此可能会不准确。

<img src="../../pic/235.png" style="zoom:80%;" />

上面的critic都是只将state做为输入，接下来要引入Q-Learning的critic了：

Q function将state和action作为输入，对应的分数作为输出。它考虑的是某个state在采取某个action之后得到的分数。注意，与之前的critic不同的是，Q function固定了第一次的action，第二次及其之后的action没有固定，而之前的critic没有固定所有的action。

<img src="../../pic/236.png" style="zoom:80%;" />

那么考虑TD方式的Q-Learning，那么在架构中有两个network，为了方便训练，引入evaluation network和target network，对target network进行固定，它提供$r+Q^\pi(s_{t+1},\pi(s_{t+1}))$，而evaluation network是实时更新的，在evaluation network更新N次后，将target network更新为evaluation network的最新状态。

<img src="../../pic/237.png" style="zoom:80%;" />

以下是Q-Learning的流程：

<img src="../../pic/238.png" style="zoom:80%;" />

## Smart Resource Allocation for Mobile Edge Computing: A Deep Reinforcement Learning Approach

期刊：IEEE Transactions on Emerging Topics in Computing

### 摘要

提出了一个智能的、基于深度强化学习的资源分配（DRLRA）方案，该方案能够在不同的MEC环境下自适应地分配计算和网络资源，减少平均服务时间，平衡资源的使用。实验结果表明，该算法在可变MEC条件下的性能优于传统OSPF算法。

### 模型

网络资源状态的方差：

![](../../pic/222.png)

其中，$l_{e_i}^{net}$表示在datalink`e_i`上的资源量，$|E|$为datalink的总数。datalink指MEC之间的链路。

计算资源分配的方差：

![](../../pic/223.png)

其中，$l_v^{cp}$表示MEC $v$上的计算资源数量，$|V|$表示MEC总数。

网络资源状态我认为指的是链路的带宽等，即链路的本身状态，而计算资源就是指MEC上的CPU等状态。

上面两个指标用于后面的DQN（深度强化学习网络）的Reward的计算。用这两个指标衡量整个网络的“健康”程度（方差大那么意味着可能存在单点问题，即一部分的MEC和链路有特别大的压力，然而其余部分的压力却很少）。

以下为DQN的框架示意图：

![](../../pic/224.png)

这篇文章更改了DQN的损失函数，使用均方误差：

![](../../pic/225.png)

其中，$r+\gamma max_{a_{t+1}}Q'(s_{t+1},a_{t+1};\omega^-)$是由target network计算出来的，$Q(s_t,a_t;\omega)$是由evaluation network计算出来的。

初始化时两个network的$\omega$相等，他们之后的更新频率不同。evaluation network每次都更新，而target network在evaluation network更新n轮后才会更新一次，更新操作就是直接将evaluation network的$\omega$赋值给target network。

接下来是DQN的state、action、reward的定义：

+ state：$s=\{p_v^m,\forall v\in V,\forall m\in M\}$，$p_v^m$表示对于application m的请求在MEC v上的个数。（可以理解为某个MEC上对某个服务的请求的数量）。
+ action：$a=\{\alpha_v^m,\forall v\in V,\forall m\in M\}$，$\alpha_v^m$表示MEC v上的、对于applicatoin m的所有请求采取的action。采取的action可以是向相邻的MEC转发请求。
+ reward：$r=\{\omega_1\cdot T+\omega_2\cdot b^{net}+ \omega_3\cdot b^{cp}\}$，其中，$T$为所有请求的响应时间，$b^{net}$、$b^{cp}$分别可以由之前提到的两个指标$var(l^{net})$、$var(l^{cp})$来计算，分别表示网络链路状态的平衡、网络计算资源状态的平衡。$\omega$可调节的权重。

以下为算法流程图：

<img src="../../pic/226.png" style="zoom:80%;" />

<img src="../../pic/227.png" style="zoom:80%;" />

<img src="../../pic/228.png" style="zoom:80%;" />

### 实验

网络拓扑结构参照了Topology Zoo，实验对照只有一个算法开放式最短路径优先（Open Shortest Path First (OSPF)）。

讨论了在application数量不同、MECS的处理能力不同、request聚集情况不同、application的计算需求不同的情况下，两个算法的平均相应时长的比较情况：

<img src="../../pic/229.png" style="zoom:80%;" />

<img src="/home/yikang/Document/gitRep/Note/pic/230.png" style="zoom:80%;" />

### 小结

这篇文章完全基于裸的DQN框架做的，只是对state、actioin、reward进行了设计，同时使用了均方差作为损失函数（这篇文章也没有说为什么）。

同时state、action、reward等只是做了粗略的说明，没有透露细节。

不过对于reward的设计还是很新颖的，用方差来表示整个网络的计算资源的平衡。

## Survey of Personalization Techniques for Federated Learning

期刊：arXiv

### 摘要



## Personalized Federated Learning with Moreau Envelopes

期刊：arXiv

### 摘要

本文提出了一种个性化联邦学习方案，该方案引入了基于客户端损失函数的 Moreau envelopes 优化。通过该方案，客户端不仅可以像经典联邦学习一样构建全局模型，而且可以利用全局模型来优化其个性化模型。从几何的角度分析，该方案中的全局模型可以看作是所有客户端一致同意的“中心点”，而个性化模型是客户端根据其异构数据分布来构建的遵循不同方向的点。

### 模型

传统的FedAvg算法在将各个client的权重w aggragate后生成加权平均后的global w，然后将这个w作为所有client的新一轮的模型。

这篇文章与传统的FedAvg不同，并没有将这个global w直接作为所有client的新一轮模型，而是保留所有client的local w信息，即仍用本地的模型进行训练，不同的是，损失函数新增了一项Moreau envelopes，如下：

<img src="../../pic/216.png" style="zoom:80%;" />

这个新项用于保留本地的个性化模型，同时向所有client的中心点靠拢。

因此整个模型为：

<img src="../../pic/217.png" style="zoom:80%;" />

算法流程：

<img src="../../pic/218.png" style="zoom:80%;" />

（7）中的$h$函数为：

<img src="../../pic/231.png" style="zoom:80%;" />

（8）中的梯度更新使用了近似权重：

<img src="../../pic/219.png" style="zoom:80%;" />

这篇文章对这个近似权重的收敛性做出了证明。

### 实验

实验对照的是经典的FedAvg以及基于元学习的Per-FedAvg，Per-FedAvg的模型如下：

<img src="../../pic/220.png" style="zoom:80%;" />

Per-FedAvg就是元学习在FL下的变式。因为这篇文章提出的pFedMe算法是个性化算法，而元学习也是个性化算法，因此将它作为实验的对照加了进来。

使用的是MNIST数据集，是一个手写数字数据集，作者将作者将完整的 MNIST 数据集分发给 N=20 个客户端。为了根据本地数据大小和类别对异构设置进行建模，每个客户端都被分配了一个不同的本地数据大小。

作者对 pFedMe、FedAvg 和 Per-FedAvg 进行了比较。MNIST 数据集中的实验结果见图 4。pFedMe 的个性化模型在强凸设置下的准确率分别比其全局模型 Per-FedAvg 和 FedAvg 高 1.1%、1.3% 和 1.5%。非凸设置下的相应数据为 0.9%、0.9% 和 1.3%。

![](../../pic/221.png)

### 小结

本文提出了一种个性化联邦学习方法 pFedMe。pFedMe 利用了 Moreau envelope 函数，该函数有助于将个性化模型优化从全局模型学习中分解出来，从而使得 pFedMe 可以类似于 FedAvg 更新全局模型，但又能根据每个客户端的本地数据分布进行优化，得到个性化模型。

