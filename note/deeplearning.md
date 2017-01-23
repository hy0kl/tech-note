# Deep Learnig

## 术语

- 边缘概率分布 marginal probability
- 求和法则 sum rule
- 干预查询 intervention query
- 因果模型 causal modeling
- 期望值 expected value
- 方差 variance
- 标准差 standard deviation 方差的平方根被称为标准差
- 协方差 covariance
- 相关系数 correlation
- Bernoulli 分布 Bernoulli distribution
- multinoulli 分布 multinoulli distribution
- 高斯分布 Gaussian distribution, 正态分布 normal distribution
- 指数分布 exponential distribution
- Laplace 分布 Laplace distribution
- 经验分布 empirical distribution
- 目标函数 objective function
- 代价函数 cost function
- 损失函数 lost function
- 误差函数 error function
- 梯度下降 gradient descent
- 学习速率 learning rate
- 权重 weight
- 偏置 bias
- 欠拟合 underfitting 发生于模型不能在训练集上获得足够低的误差
- 过拟合 overfitting 发生于训练误差和测试误差之间的差距过大
- 表示容量 representational capacity
- 有效容量 effective capacity
- 最近邻回归 nearest nighbor regression
- 贝叶斯误差 Bayes error
- 权重衰减 weight decay
- 正则化项 regularizer
- 验证集 validation set
- 渐近无偏 asymptotically unbiased
- 样本方差 sample variance
- 几乎必然收敛 almost sure convergence
- 先验概率分布 prior probability distribution
- 最大后验(Maximum A Posteriori, MAP)点估计
- 支持向量机 support vector machine, SVM
- 核技巧 kernel trick
- 径向基函数(raidal basis function, RBF)
- 随机梯度下降 stochastic gradient descent, SGD
- 激活函数 activation function
- 反向传播 back propagation
- 前向传播 forward propagation
- 异方差 heteroscedastic
- 多峰回归 multimodal regression
- 梯度截断 clip gradient
- 绝对值整流 absolute value rectification
- 渗漏整流线性单元 Leaky ReLU
- 参数化整流线性单元 parametric ReLU
- 灾难遗忘 catastrophic forgetting
- 径向基函数 radial basis function, RBF
- 硬双曲正切函数 hard tanh
- 通用近似定理 universal approximation theorem
- 计算图 computational graph
- 符号表达式 symbolic representation
- 数值 numeric value
- 并行分布式处理 Parallel Distributed Processing
- 稀疏激活 sparse activation
- 特征选择 feature selection
- 标签平滑 label smoothing
- 参数共享 parameter sharing
- 对抗样本 adversarial example
- 对抗训练 adversarial training
- 虚拟对抗样本 virtual adversarial example
- 切面距离 tangent distance
- 正切传播 tangent prop
- 经验风险 expirical risk
- 经验风险最小化 expirical risk minimization
- 替代损失函数 surrogate loss function
- 模型可辨认性 model identifiability
- 权重空间对称性 weight space symmetry
- 梯度消失（或弥散）问题 vanishing gradient problem
- 梯度爆炸问题 exploding gradient problem
- 标准初始化 normalized initialization
- 稀疏初始化 sparse initialization

## 随记

- 度量模型性能的一种方法是计算模型在测试集上的`均方误差`(mean squared error)
- 在先前未观测到的输入上表现良好的能力被称为`泛化`(generalization)
- 高斯均值参数的常用估计量被称为`样本均值`(sample mean)
- 一种控制训练算法容量的方法是选择`假设空间`(hypothesis space),即能够选为解决方案的学习算法函数集
- 当数据的维数很高时,很多机器学习问题变得相当困难.这种现象被称为`维数灾难`(curse of dimensionality)

## 深度前馈网络

`深度前馈网络`(deep feedforward network),也叫作`前馈神经网络`(feedforward neural network)或者`多层感知机`(multilayer perceptron, MLP),是典型的深度学习模型.

整流线性激活函数是被推荐用于大多数前馈神经网络的默认激活函数.

`泛函`(functional)是函数到实数的映射.

带有将Gaussian混合作为其输出的神经网络通常被称为`混合密度网络`(mixture density network).

神经网络设计的另一个关键点是确定它的结构.`结构`(architecture)一词是指网络的整体结构:它应该具有多少单元,以及这些单元应该如何连接.

动态规划 dynamic programming

### 深度学习界以外的微分
深度学习界在某种程序上已经与更广泛的计算机科学界隔离开来,并且在很大程序上发展了自己关于如何进行微分的文化态度.更一般地,`自动微分`(automatic differentiation)领域关心如何以算法方式计算导数.这里描述的反射传播算法只是自动微分的一种方法.它是一种称为`反向模式累加`(reverse mode accumulation)的更广泛类型的技术的特殊情况.

代替明确地计算Hessian矩阵,典型的深度学习方法是使用`Krylov方法`(Krylov method).Krylov 方法是用于执行各种操作的一组迭代技术,这些操作包括像近似求解矩阵的逆、或者近似矩阵的特征值或特征向量等，而不使用矩阵-向量乘法以外的任何操作。

## 深度学习的正则化

`提前终止`（early stopping）

目前为止，最流行和广泛使用的参数共享出现在应用于计算机视觉的`卷积神经网络（CNN）`。

权重衰减施加直接作用于模型参数的惩罚。另一种策略是将惩罚放在神经网络的激活单元，鼓励对应的激活是稀疏。

`正交匹配追踪`(orthogonal matching pursuit)

`Bagging`(bootstrap aggregating)是通过结合几个模型降低泛化误差的技术。主要想法是分别训练几个不同的模型，然后让所有模型表决测试样例的输出。这是机器学习中常规策略的一个例子，被称为`模型平均`（model averaging）。采用这种策略的技术被称为集成方法。

模型平均是减少泛化误差一个非常强大可靠的方法。

`权重比例推断规则`(weight scaling inference rule)。

## 深度模型中的优化

将机器学习问题转化成优化问题的最简单方法是最小化训练集上的期望损失。

`AdaGrad`算法

`RMSProp`算法

`Adam`(Adaptive moments)算法

共轭梯度是一种通过迭代下降的`共轭方向（conjugate directions）`以有效避免Hessian矩阵求逆计算的方法。

`Broyden-Fletcher-Goldfarb-Shanno(BFGS)`算法

`batch normalization`

`坐标下降`(coordinate descent) `块坐标下降`(block coordinate descent)

`Polyak平均`

训练简单模型求解简化问题的方法统称为`预训练`(pretraining)。

`贪心算法`(greedy algorithm)

`贪心监督预训练`(greedy supervised pretraining)

在实践中，选择一族容易优化的模型比使用一个强大的优化算法更重要。

`连续方法`(continuation method)是一族通过挑选初始点使优化更容易的方法，以确保局部优化花费大部分时间在表现良好的空间。

## 卷积神经网络

`卷积网络`(convolutional network)，也叫做`卷积神经网络`(convergence neural network, CNN)，是一种专门用来处理具有类似风格结构的数据的神经网络。“卷积网络”一词表明该网络使用了`卷积`(convolution)这种数学运算。卷积是种特殊的线性运算。卷积网络是指那些至少在网络的一层中使用卷积运算来替代一般的矩阵乘法运算的神经网络。


