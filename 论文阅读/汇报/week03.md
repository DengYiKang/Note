# week03

## An Efficient Framework for Clustered Federated Learning

期刊：arXiv

年份：2020

### 摘要

提出了一种新的迭代联邦聚类算法（IFCA），该算法通过梯度下降交替估计用户的聚类身份，并优化用户聚类的模型参数。分析了该算法在具有平方损失的线性模型中的收敛速度，然后分析了一般强凸光滑损失函数的收敛速度。

### 模型

<img src="../../pic/288.png" style="zoom:80%;" />

大致的思想是：首先确定簇的个数k，中心服务器生成k个初始权重$\theta_j^{(0)}$，将这k个发给所有的client。client进行对这些权重的挑选，选择一个能使本地的loss function最低的权重$\theta_i^{(0)}$，那么这个client就属于第$i$个簇，这一轮的global update就只与第$i$个簇的所有client进行FL。每一轮的clustering都可能不一样。

<img src="../../pic/289.png" style="zoom:80%;" />