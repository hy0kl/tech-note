# 绪论
## 编码标准的话题包括:
- [PHP File 文件格式](#php-file-文件格式)
- [命名约定](#命名约定)
- [编码风格](#编码风格)
- [注释文档](#注释文档)
- [其他](#其他)

### 目标
编码标准对任何开发项目都很重要，特别是很多开发者在同一项目上工作。编码标准帮助确保代码的高质量、少 bug 和容易维护。

#PHP File 文件格式


## 常规
对于只包含有 PHP 代码的文件，结束标志("?>")是不允许存在的，PHP 自身不需要("?>"), 这样做, 可以防止它的末尾的被意外地注入相应。

文件的编码类型需要统一,如果没有特殊说明,全部统一为 utf-8.

## 缩进
缩进由四个空格组成，禁止使用制表符 TAB 。

## 行的最大长度
一行 80 字符以内是比较合适，就是说，开发者应当努力在可能的情况下保持每行代码少于 80 个字符，在有些情况下，长点也可以, 但最多为 120 个字符。

## 行结束标志
行结束标志遵循 Unix 文本文件的约定，行必需以单个换行符（LF）结束。换行符在文件中表示为 10，或16进制的 0x0A。

***注：不要使用 苹果操作系统的回车（0x0D）或 Windows 电脑的回车换行组合如（0x0D,0x0A）。***

##命名约定

### 函数和方法
函数名只包含字母数字字符，下划线是不允许的。数字是允许的但大多数情况下不鼓励。

函数名总是以小写开头，当函数名包含多个单词，每个子的首字母必须大写，这就是所谓的 “驼峰” 格式。

一般鼓励使用冗长的名字，函数名应当长到足以说明函数的意图和行为。

这些是可接受的函数名的例子：

```
filterInput()
getElementById()
widgetFactory()
```
对于面向对象编程，实例或静态变量的访问器总是以 "get" 或 "set" 为前缀。在设计模式实现方面，如单态模式（singleton）或工厂模式（factory）， 方法的名字应当包含模式的名字，这样名字更能描述整个行为。

在对象中的方法，声明为 "private" 或 "protected" 的， 名称的首字符必须是一个单个的下划线，这是唯一的下划线在方法名字中的用法。声明为 "public" 的从不包含下划线。

全局函数 (如："floating functions") 允许但大多数情况下不鼓励，建议把这类函数封装到静态类里。

### 变量
变量只包含数字字母字符，大多数情况下不鼓励使用数字，风格要统一,要么采用标准驼峰式,要采用 C 下划线式。

声明为 "private" 或 "protected" 的实例变量名必须以一个单个下划线开头，这是唯一的下划线在程序中的用法，声明为 "public" 的不应当以下划线开头。

对函数名一样，变量名总以小写字母开头并遵循“驼峰式”命名约定,或下划线分隔提高可读性。

一般鼓励使用冗长的名字，这样容易理解代码，开发者知道把数据存到哪里。除非在小循环里，不鼓励使用简洁的名字如 "$i" 和 "$n" 。如果一个循环超过 20 行代码，索引的变量名必须有个具有描述意义的名字。

### 常量
常量包含数字字母字符和下划线，数字允许作为常量名。

常量名的所有字母必须大写。

常量中的单词必须以下划线分隔，例如可以这样

EMBED_SUPPRESS_EMBED_EXCEPTION 但不许这样
EMBED_SUPPRESSEMBEDEXCEPTION。

常量必须通过 "const" 定义为类的成员，强烈不鼓励使用 "define" 定义的全局常量。

#编码风格

## PHP 代码划分（Demarcation）
PHP 代码总是用完整的标准的 PHP 标签定界:

```php
<?php
/** some code here ... */
?>
```
短标签 <?=?> 是不允许的，只包含 PHP 代码的文件，不要结束标签。

## 字符串
### 字符串文字
当字符串是文字(不包含变量)，应当用单引号（ apostrophe ）来括起来：

```php
$a = 'Example String';
```
包含单引号（'）的字符串文字

当文字字符串包含单引号（apostrophe ）就用双引号括起来，特别在 SQL 语句中有用：

```php
$sql = "SELECT `id`, `name` from `people` WHERE `name`='Fred' OR `name`='Susan'";
```
在转义单引号时，上述语法是首选的，因为很容易阅读。

## 变量替换
### 变量替换有下面这些形式：

```php
$greeting = "Hello $name, welcome back!";
$greeting = "Hello {$name}, welcome back!";
// 为保持一致，这个形式不允许：
$greeting = "Hello ${name}, welcome back!";
```

## 字符串连接
字符串必需用 "." 操作符连接，在它的前后加上空格以提高可读性：

```php
$company = 'Zend' . ' ' . 'Technologies';
```
当用 "." 操作符连接字符串，鼓励把代码可以分成多个行，也是为提高可读性。在这些例子中，每个连续的行应当由 whitespace 来填补，例如 "." 和 "=" 对齐：

```php
$sql = "SELECT `id`, `name` FROM `people` "
    . "WHERE `name` = 'Susan' "
    . "ORDER BY `name` ASC ";
```

# 数组
## 数字索引数组
索引不能为负数

建议数组索引从 0 开始。

当用 array 函数声明有索引的数组，在每个逗号的后面间隔空格以提高可读性：

```php
$sampleArray = array(1, 2, 3, 'Zend', 'Studio');
```
可以用 "array" 声明多行有索引的数组，在每个连续行的开头要用空格填补对齐：

```php
$sampleArray = array(
    1, 2, 3, 'Zend', 'Studio',
    $a, $b, $c,
    56.44, $d, 500
);
```

## 关联数组
当用声明关联数组，array 我们鼓励把代码分成多行，在每个连续行的开头用空格填补来对齐键和值：

```php
$sampleArray = array(
    'firstKey'  => 'firstValue',
    'secondKey' => 'secondValue',
);
```

# 类
## 类的声明
花括号应当从类名下一行开始(the "one true brace" form)。

每个类必须有一个符合 PHPDocumentor 标准的文档块。

类中所有代码必需用四个空格的缩进。

每个 PHP 文件中只有一个类。

放另外的代码到类里允许但不鼓励。在这样的文件中，用两行空格来分隔类和其它代码。

下面是个可接受的类的例子： // 459 9506 － 441 9658 下次从这里开始

```php
/**
 * Documentation Block Here
 */
class SampleClass
{
    // 类的所有内容
    // 必需缩进四个空格
}
```

## 类成员变量
变量的声明必须在类的顶部，在方法的上方声明。

不允许使用 var ，要用 private、 protected 或 public。 直接访问 public 变量是允许的但不鼓励，最好使用访问器 （set/get）。

## 函数和方法
### 函数和方法声明
在类中的函数必须用 private、 protected 或 public 声明它们的可见性。

象类一样，花括号从函数名的下一行开始(the "one true brace" form)。

函数名和括参数的圆括号中间没有空格。

强烈反对使用全局函数。

下面是可接受的在类中的函数声明的例子：

```php
/**
 * Documentation Block Here
 */
class Foo
{
    /**
     * Documentation Block Here
     */
    public function bar()
    {
        // 函数的所有内容
        // 必需缩进四个空格
    }
}
// 注： 传址（Pass-by-reference）是在方法声明中允许的唯一的参数传递机制。
/**
 * Documentation Block Here
 */
class Foo
{
    /**
     * Documentation Block Here
     */
    public function bar(&$baz)
    {}
}
// 传址在调用时是严格禁止的。
// 返回值不能在圆括号中，这妨碍可读性而且如果将来方法被修改成传址方式，代码会有问题。
/**
 * Documentation Block Here
 */
class Foo
{
    /**
     * WRONG
     */
    public function bar()
    {
        return($this->bar);
    }

    /**
     * RIGHT
     */
    public function bar()
    {
        return $this->bar;
    }
}
```

### 函数和方法的用法
函数的参数应当用逗号和紧接着的空格分开，下面可接受的调用的例子中的函数带有三个参数：
    threeArguments(1, 2, 3);

传址方式在调用的时候是严格禁止的，参见函数的声明一节如何正确使用函数的传址方式。

带有数组参数的函数，函数的调用可包括 "array" 提示并可以分成多行来提高可读性，同时，书写数组的标准仍然适用：

```php
threeArguments(array(1, 2, 3), 2, 3);
threeArguments(array(1, 2, 3, 'Zend', 'Studio',
                    $a, $b, $c,
                    56.44, $d, 500), 2, 3);
```

# 控制语句
## if/else/elseif
使用 if and elseif 的控制语句在条件语句的圆括号前后都必须有一个空格。

在圆括号里的条件语句，操作符必须用空格分开，鼓励使用多重圆括号以提高在复杂的条件中划分逻辑组合。

前花括号必须和条件语句在同一行，后花括号单独在最后一行，其中的内容用四个空格缩进。

```php
if ($a != 2) {
    $a = 2;
}
```
对包括"elseif" 或 "else"的 "if" 语句，和 "if" 结构的格式类似， 下面的例子示例 "if" 语句， 包括 "elseif" 或 "else" 的格式约定：

```php
if ($a != 2) {
    $a = 2;
} else {
    $a = 7;
}

if ($a != 2) {
    $a = 2;
} elseif ($a == 3) {
    $a = 4;
} else {
    $a = 7;
}
```
在有些情况下， PHP 允许这些语句不用花括号，但在代码标准里，它们（"if"、 "elseif" 或 "else" 语句）必须使用花括号。
"elseif" 是允许的但强烈不鼓励，我们支持 "else if" 组合。

## switch
在 "switch" 结构里的控制语句在条件语句的圆括号前后必须都有一个单个的空格。

"switch" 里的代码必须有四个空格缩进，在"case"里的代码再缩进四个空格。

```php
switch ($numPeople) {
    case 1:
        break;

    case 2:
        break;

    default:
        break;
}
```
switch 语句应当有 default。

- 注： 有时候，在 falls through 到下个 case 的 case 语句中不写 break or return 很有用。 为了区别于 bug，任何 case 语句中，所有不写 break or return 的地方应当有一个 "// break intentionally omitted" 这样的注释来表明 break 是故意忽略的。

#注释文档

## 格式
所有类文件必须在文件的顶部包含文件级 （"file-level"）的 docblock ，在每个类的顶部放置一个 "class-level" 的 docblock。下面是一些例子：

## 文件
每个包含 PHP 代码的文件必须至少在文件顶部的 docblock 包含这些 phpDocumentor 标签：

```php
/**
 * 文件的简短描述
 *
 * 文件的详细描述（如果有的话）... ...
 *
 * LICENSE: 一些 license 信息
 *
 * @copyright  Copyright (c) 2005-2011 Zend Technologies USA Inc. (http://www.zend.com)
 * @license    http://framework.zend.com/license/3_0.txt   BSD License
 * @version    $Id:$
 * @link       http://framework.zend.com/package/PackageName
 * @since      File available since Release 1.5.0
*/
```

## 类
每个类必须至少包含这些 phpDocumentor 标签：

```php
/**
 * 类的简述
 *
 * 类的详细描述 （如果有的话）... ...
 *
 * @copyright  Copyright (c) 2005-2011 Zend Technologies USA Inc. (http://www.zend.com)
 * @license    http://framework.zend.com/license/   BSD License
 * @version    Release: @package_version@
 * @link       http://framework.zend.com/package/PackageName
 * @since      Class available since Release 1.5.0
 * @deprecated Class deprecated in Release 2.0.0
 */
```

## 函数
每个函数，包括对象方法，必须有最少包含下列内容的文档块（docblock）：

- 函数的描述
- 所有参数
- 所有可能的返回值

因为访问级已经通过 "public"、 "private" 或 "protected" 声明， 不需要使用 "@access"。
如果函数/方法抛出一个异常，使用 @throws 于所有已知的异常类：

```
    @throws exceptionclass [description]
```

#其他

- [约定编码规范](#约定编码规范)
- [编码小技巧](#编码小技巧)
- [常见的坑](#常见的坑)

<h2 id="standard">约定编码规范</h2>

1. 函数调用时将函数名与左(之间不要留空白,关键字与左(之间留空白.
2. 二元操作符左右都留空.
3. 函数参数之间的,右边留空.
4. if/else 条件分支中,哪怕只有一行代码,也应该用 {} 来分块.
5. 开发环境将错误报告设置到最高,有效扼制低级 bug.

```php
$pos = strpos($uri, '?');

if ('admin' == $user)
{
    print('hello!');
}
else
{
    echo 'sorry';
}
```

##编码小技巧

1. 如果 if 的条件判断是变量和常量相比较,建议将常量写在 == 的左边,可以有效防止少写一个 = 带来的隐患.
2. 字符串连接时,如果明确没有变量替换,则优先使用单引号 ''.
3. 尽量使用 foreach 遍历数组.
4. 字符串输出使用 echo 代替 print. echo 是个关键字, print() 是个函数,有返回值,有函数调用的堆栈时间.
5. strlen() 的时间复杂是常数级的,切记不要放到 for() 的条件判断中,而是采用 *代码示例 1*:
6. count() 原理同上.
7. [] 要比 array_push() 效率高.
8. 对数组的操作,原生 API 效率很高,优先考虑使用.例如对元素去重的 array_uniq();
9. 在数组中查找元素, isset() 比 in_array() 效率要高.但要注意的是,如果数组元素被赋值为 null, isset() 返回真值,但也许不符合调用者预期.
10. 对象中的独立方法尽量设计为静态方法.
11. 编码转换时,用 mb_ 函数库代替 iconv* 函数库.
12. 将逻辑相关性相比较强的代码和其他代码以空行来分块,有效提高代码可读性.毕竟代码读的时间远比写的时间长得多.

*代码示例 1*

```php
$len = strlen($str);
for ($i = 0; $i < $len; $i++)
{
    ;
}
```

##常见的坑

1). 遍历数组时,切不可用采用引用的方式,例如下面的代码,应引起足够的重视:

```php
foreach ($users as &$info)
{
    /** do something ... */
    $info['foo'] = 'bar';
}
```
很多采用"引用"遍历的同学,都为方便在遍历的同时修改原数组的数据.应该采用如下的方式来代替:

```php
foreach ($users as $uid => $info)
{
	$users[$id]['foo'] = 'bar';
}
```

2). htmlspecialchars() 函数有陷井.它的原型如下:

```php
string htmlspecialchars ( string $string
    [, int $flags = ENT_COMPAT | ENT_HTML401 [, string $encoding = 'UTF-8'
    [, bool $double_encode = true ]]] )
```
它默认不对 ' 引号进行 html 转义,有可能对过滤不严格的代码造成 sql 注入.需要指定第二个参数为 *ENT_QUOTES* 才会将双引号 " 和单引号 ' 转义成对应的 html 实体.

3). strip_tags() 在过滤不规范 html 时存在陷井,存在过滤不符合预期的情况,可能会打断文档流.

4). 三元操作符的优先级问题,适当的 () 可有效解决,并且提高代码可读性.
