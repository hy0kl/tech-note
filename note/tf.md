# TensorFlow机器学习实战指南

[原书](https://github.com/nfmcclure/tensorflow_cookbook)

# 第1章

```
pip3 install ipython
pip3 install tensorflow
pip3 install numpy
pip3 install matplotlib
pip3 install scipy
pip3 install keras
```

## 通用模式

1. 导入/生成样本数据集
1. 转换和归一化数据
1. 划分样本数据集为训练样本集、测试样本集和验证样本集
1. 设置机器学习参数（超参数）
1. 初始化变量和占位符
1. 定义模型结构
1. 声明损失函数
1. 初始化模型和训练模型
1. 评估机器学习模型
1. 调优超参数
1. 发布/预测结果

# 其他

## 线性回归 cost/lost function

$$ cost(W,b) = \frac{1}{m}\sum_{i=1}^{m}(H(x^{(i)})-y^{(i)})^{2} $$

`hypothesis = X * W + b`

`cost = tf.reduce_mean(tf.squre(hypothesis - Y))`

## 逻辑回归

$$ H(X) = sigmoid(XW) = \frac{1}{1 + e^{-XW}} $$

`hypothesis = tf.sigmoid(tf.matmul(X, W) + b)`

$$ cost(W) = -\frac{1}{m}\sum ylog(H(x)) + (1 -y)(log(1 - H(x))) $$

`cont = -tf.reduce_mean(Y * tf.log(hypothesis) + (1 - Y) * tf.log(1 - hypothesis))`

## Softmax Classification

`hypothesis = tf.nn.softmax(tf.matmul(X, W) + b)`

## Cost function: cross entropy

`cost = tf.reduce_mean(-tf.reduce_sum(Y * tf.log(hypothesis), axis=1))`

## Neural Networks


