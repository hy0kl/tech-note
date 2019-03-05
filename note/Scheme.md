# 将Scheme用作计算器

## 算术操作

- 函数`exact->inexact`用于把分数转换为浮点数。
- 函数`quotient`用于求`商数（quotient）`
- 函数`remainder`和`modulo`用于求余数`（remainder）`
- 函数`sqrt`用于求参数的平方根`（square root）`
- 使用`integer->char`可以将整数转化为字符。

## 常用函数

- 函数`string->list`可以将字符串转化为由字符构成的表。

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

## 如何定义简单函数并加载它们

可以使用`define`来将一个符号与一个值绑定。通过这个操作符定义例如`数`、`字符`、`表`、`函数`等任何类型的全局参数。

操作符`define`用于声明变量，它接受两个参数。`define`运算符会使用第一个参数作为全局参数，并将其与第二个参数绑定起来。

特殊形式`lambda`用于定义过程。`lambda`需要至少一个的参数，第一个参数是由定义的过程所需的参数组成的表。

## 定义有参数的函数

预定义函数`string-append`可以接受任意多个数的参数，并返回将这些参数连结在一起后的字符串。

# 分支

## if表达式

- `(if predicate then_value else_value)`
- `true`是除`false`以外的任意值，`true`使用`#t`表示，`false`用`#f`表示。
- 使用函数`null?`来判断表是否为空。
- 函数`not`可用于对谓词取反。
- `then_value`和`else_value`都应该是S-表达式。

## and和or

`and`和`or`是用于组合条件的两个特殊形式。Scheme中的`and`和`or`不同于C语言中的约定。它们不返回一个布尔值（`#t`或`#f`），而是返回给定的参数之一。

### and

`and`具有任意个数的参数，并从左到右对它们求值。如果某一参数为`#f`，那么它就返回`#f`，而不对剩余参数求值。反过来说，如果所有的参数都不是`#f`，那么就返回最后一个参数的值。

### or

`or`具有可变个数的参数，并从左到右对它们求值。它返回第一个不是值`#f`的参数，而余下的参数不会被求值。如果所有的参数的值都是`#f`的话，则返回最后一个参数的值。

## cond表达式

```
(cond
  (predicate_1 clauses_1)
  (predicate_2 clauses_2)
    ......
  (predicate_n clauses_n)
  (else        clauses_else))
```

## 做出判断的函数

### `eq?`、`eqv?`和`equal?`

基本函数`eq?`、`eqv?`、`equal?`具有两个参数，用于检查这两个参数是否“一致”。

- `eq?` 该函数比较两个对象的地址，如果相同的话就返回`#t`。不要使用`eq?`来比较数字
- `eqv?` 该函数比较两个存储在内存中的对象的类型和值。如果类型和值都一致的话就返回`#t`。不能用于类似于表和字符串一类的序列比较
- `equal?` 该函数用于比较类似于表或者字符串一类的序列。

## 用于检查数据类型的函数

- `pair?` 如果对象为序对则返回`#t`；
- `list?` 如果对象是一个表则返回`#t`。要小心的是空表`'()`是一个表但是不是一个序对。
- `null?` 如果对象是空表`'()`的话就返回`#t`。
- `symbol?` 如果对象是一个符号则返回`#t`。
- `char?` 如果对象是一个字符则返回`#t`。
- `string?` 如果对象是一个字符串则返回`#t`。
- `number?` 如果对象是一个数字则返回`#t`。
- `complex?` 如果对象是一个复数则返回`#t`。
- `real?` 如果对象是一个实数则返回`#t`。
- `rational?` 如果对象是一个有理数则返回`#t`。
- `integer?` 如果对象是一个整数则返回`#t`。
- `exact?` 如果对象不是一个浮点数的话则返回`#t`。
- `inexact?` 如果对象是一个浮点数的话则返回`#t`。

- `zero?`
- `positive?` 是否为正数
- `negative?`

## 用于比较数的函数

`=`、`>`、`<`、`<=`、`>=`

## 用于比较符号的函数

在比较字符的时候可以使用`char=?`、`char<?`、`char>?`、`char<=?`以及`char>=?`函数。

## 用于比较字符串的函数

比较字符串时，可以使用`string=?`和`string-ci=?`等函数。

# 局部变量

## let表达式

使用`let`表达式可以定义局部变量。

`(let binds body)`

变量的`作用域（Scope）`为`body`体，也就是说变量只在`body`中有效。

`let*`表达式可以用于引用定义在同一个绑定中的变量。实际上，`let*`只是嵌套的`let`表达式的语法糖

实际上，`let`表达式只是`lambda`表达式的一个语法糖

`词法闭包（lexical closure）`

# 重复

## 递归

在自己的定义中调用自己的函数叫做`递归函数（Recursive Function）`。

## 尾递归

尾递归函数包含了计算结果，当计算结束时直接将其返回。特别地，由于Scheme规范要求尾递归调用转化为循环，因此尾递归调用就不存在函数调用开销。

## 命名let

命名`let（named let）`可以用来表达循环。

## letrec

`letrec`类似于`let`，但它允许一个名字递归地调用它自己。语法`letrec`通常用于定义复杂的递归函数。

语法`letrec`是定义局部变量的俗成方式。

## do表达式

```
(do binds (predicate value)
    body)
```

变量在`binds`部分被绑定，而如果`predicate`被求值为真，则函数从循环中`逃逸（escape）`出来，并返回值`value`，否则循环继续进行。

# 高阶函数

`高阶函数（Higher Order Function）`是一种以函数为参数的函数。它们都被用于`映射（mapping）`、`过滤（filtering）`、`归档（folding）`和`排序（sorting）`表。

## 映射

### map

```
(map procedure list1 list2 ...)
```

### for-each

### 过滤

MIT-Scheme实现提供了`keep-matching-items`和`delete-matching-item`两个函数。

### 归档

MIT-Scheme提供了`reduce`等函数

### 排序

MIT-Scheme提供了`sort`（实为`merge-sort`实现）和`quick-sort`函数。

### apply函数

`apply`函数是将一个过程应用于一个表（译注：将表展开，作为过程的参数）。此函数具有任意多个参数，但首参数和末参数分别应该是一个过程和一个表。

# 输入输出

## 从文件输入

`open-input-file`，`read-char`和`eof-object?`

## 语法call-with-input-file和with-input-from-file

## read

函数`(read port)`从端口`port`中读入一个S-表达式。

## 输出至文件

`(open-output-file filename)`

`(close-output-port port)`

`(call-with-output-file filename procedure)`

`(with-output-to-file filename procedure)`

## 用于输出的函数

`(write obj port)`

`(display obj port)`

`(newline port)`

`(write-char char port)`

# 赋值

## set!

`set!`可以为一个参数赋值。

## 赋值和内部状态

### 静态作用域（词法闭包）

作用域与源代码书写方式一致的作用域称为`“词法闭包（Lexical closure）”`或`“静态作用域（Static scope）”`。

特殊形式`let`、`lambda`、`letrec`生成闭包。`lambda`表达式的参数仅在函数定义内部有效。`let`只是`lambda`的语法糖，因此二者无异。

使用赋值和词法闭包来实现内部状态

`Scheme`过程的主要目的是返回一个值，而另一个目的则称为`副作用（Side Effect）`。赋值和IO操作就是副作用。

如果代码体中有多条 S-表达式，那么可以使用`begin`语句让它们成组。

## 表的破坏性操作（set-car!，set-cdr!）

## 队列

队列可以用`set-car!`和`set-cdr!`实现。队列是一种`先进先出(First in first out, FIFO)`的数据结构，表则是`先进后出(First in last out，FILO)`。

# 字符与字符串

## 字符

- 在某个字符前添加`#\`来表明该物是一个字符。
- 字符`#\Space`、`#\Tab`、`#\Linefeed`和`#\Return`分别代表`空格（Space）`、`制表符（Tab）`，`Linefeed`和`返回（Return）`。
- `(char? obj)` 如果`obj`是一个字符则返回`#t`。
- `(char=? c1 c3)` 如果`c1`和`c2`是同一个字符的话则返回`#t`。
- `(char->integer c)` 将`c`转化为对应的整数（字符代码，character code）。
- `(integer->char n)` 将一个整数转化为对应的字符。
- `(char<? c1 c2)`
- `(char<= c1 c2)`
- `(char> c1 c2)`
- `(char>= c1 c2)`

### 比较函数对大小写不敏感

- `(char-ci=? c1 c2)`
- `(char-ci<? c1 c2)`
- `(char-ci<=? c1 c2)`
- `(char-ci>? c1 c2)`
- `(char-ci>=? c1 c2)`

### 检测字符c是否为字母、数字、空白符、大写字母或小写字母

- `(char-alphabetic? c)`
- `(char-numeric? c)`
- `(char-whitespace? c)`
- `(char-upper-case? c)`
- `(char-lower-case? c)`

- `(char-upcase c)`
- `(char-downcase c)`

## 字符串

- `(string? s)` 如果`s`是一个字符则返回`#t`。
- `(make-string n c)` 返回由`n`个字符`c`组成的字符串。参数`c`可选。
- `(string-length s)` 返回字符串`s`的长度。
- `(string=? s1 s2)` 如果字符串`s1`和`s2`相同的话则返回`#t`。
- `(string-ref s idx)` 返回字符串`s`中索引为`idx`的字符（索引从0开始计数）。
- `(string-set! s idx c)` 将字符串`s`中索引为`idx`的字符设置为`c`。
- `(substring s start end)` 返回字符串`s`从`start`开始到`end-1`处的子串。
- `(string-append s1 s2 ...)` 连接两个字符串`s1`和`s2`
- `(string->list s)` 将字符串`s`转换为由字符构成的表。
- `(list->string ls)` 将一个由字符构成的表转换为字符串。
- `(string-copy s)` 复制字符串`s`。

# 符号
