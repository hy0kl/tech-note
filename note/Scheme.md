# 将Scheme用作计算器

## 算术操作

- 函数`exact->inexact`用于把分数转换为浮点数。
- 函数`quotient`用于求`商数（quotient）`
- 函数`remainder`和`modulo`用于求余数`（remainder）`
- 函数`sqrt`用于求参数的平方根`（square root）`

## 三角函数

数学上的三角函数，诸如`sin`，`cos`，`tan`，`asin`，`acos`和`atan`都可以在Scheme中使用。

## 指数和对数

指数通过`exp`函数运算，对数通过`log`函数运算。
`a`的`b`次幂可以通过`(expt a b)`来计算。

# 生成表

## Cons单元和表

`Cons单元（Cons cells）`。`Cons单元`是一个存放了两个地址的内存空间。`Cons单元`可用函数`cons`生成。

`car`和`cdr`分别是*寄存器地址部分（Contents of the Address part of the Register）*和*寄存器减量部分（Contents of the Decrement part of the Register）*的简称。这些名字最初来源于`Lisp`首次被实现所使用的硬件环境中内存空间的名字。这些名字同时也表明`Cons单元`的本质就是一个内存空间。`cons`这个名字是术语*构造（construction）*的简称。

- `#\c`代表了一个字符`c`。

## 表

表是`Cons单元`通过用`cdr`部分连接到下一个`Cons单元`的开头实现的。表中包含的`'()`被称作空表。

### 表的递归地定义：

1. `'()`是一个表
1. 如果`ls`是一个表且`obj`是某种类型的数据，那么`(cons obj ls)`也是一个表

## 引用

一个被称为`引用（quote）`的形式可以用来阻止记号被求值。它是用来将符号或者表原封不动地传递给程序，而不是求值后变成其它的东西。

`quote`的使用频率很高，他被简写为`'`。

## 特殊形式

`Scheme`有两种不同类型的操作符：其一是函数。函数会对所有的参数求值并返回值。另一种操作符则是特殊形式。特殊形式不会对所有的参数求值。除了`quote`，`lambda`，`define`，`if`，`set!`，等都是特殊形式。

## list函数

# 定义函数
