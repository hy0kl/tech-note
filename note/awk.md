# awk 的环境变量

[参考整理](http://sebug.net/paper/books/awk/)

```
变量        描述
$n          当前记录的第n个字段，字段间由FS分隔。
$0          完整的输入记录。
ARGC        命令行参数的数目。
ARGIND      命令行中当前文件的位置(从0开始算)。
ARGV        包含命令行参数的数组。
CONVFMT     数字转换格式(默认值为%.6g)
ENVIRON     环境变量关联数组。
ERRNO       最后一个系统错误的描述。
FIELDWIDTHS 字段宽度列表(用空格键分隔)。
FILENAME    当前文件名。
FNR         同NR，但相对于当前文件。
FS          字段分隔符(默认是任何空格)。
IGNORECASE  如果为真，则进行忽略大小写的匹配。
NF          当前记录中的字段数。
NR          当前记录数。
OFMT        数字的输出格式(默认值是%.6g)。
OFS         输出字段分隔符(默认值是一个空格)。
ORS         输出记录分隔符(默认值是一个换行符)。
RLENGTH     由match函数所匹配的字符串的长度。
RS          记录分隔符(默认是一个换行符)。
RSTART      由match函数所匹配的字符串的第一个位置。
SUBSEP      数组下标分隔符(默认值是\034)。
```

# awk 运算符

```
运算符                      描述
= += -= *= /= %= ^= **=     赋值
?:                          C 条件表达式
||                          逻辑或
&&                          逻辑与
~ ~!                        匹配正则表达式和不匹配正则表达式
< <= > >= != ==             关系运算符
空格                        连接
+ -                         加，减
* / &                       乘，除与求余
+ - !                       一元加，减和逻辑非
^ ***                       求幂
++ --                       增加或减少，作为前缀或后缀
$                           字段引用
in                          数组成员
```

# 命令选项

```
-F fs or --field-separator fs
    指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。

-v var=value or --asign var=value
    赋值一个用户定义变量。
```

# 数组

awk 中的数组的下标可以是数字和字母，称为关联数组。

## 下标与关联数组

- 用变量作为数组下标。如：`$ awk {name[x++] = $2;} END{for (i = 0; i < NR; i++) print i, name[i]}' test`。数组 name 中的下标是一个自定义变量 x，awk 初始化 x 的值为 0，在每次使用后增加 1。第二个域的值被赋给 name 数组的各个元素。在 END 模块中，for 循环被用于循环整个数组，从下标为 0 的元素开始，打印那些存储在数组中的值。因为下标是关健字，所以它不一定从 0 开始，可以从任何值开始。
- special for 循环用于读取关联数组中的元素。格式如下：

```
{
    for (item in arrayname) {
        print arrayname[item];
    }
}
$ awk '/^tom/{name[NR] = $1;} END{for (i in name) {print name[i];} }' test
```

打印有值的数组元素。打印的顺序是随机的。

- 用字符串作为下标。如：count["test"]
- 用域值作为数组的下标。一种新的 for 循环方式，for (index_value in array) statement。如: `$ awk '{count[$1]++} END{for (name in count) print name, count[name]}' test`。该语句将打印 $1 中字符串出现的次数。它首先以第一个域作数组 count 的下标，第一个域变化，索引就变化。
- delete 函数用于删除数组元素。如：`$ awk '{line[x++] = $1} END{for (x in line) delete(line[x])}' test`。分配给数组 line 的是第一个域的值，所有记录处理完成后，special for 循环将删除每一个元素。

# 内置函数

## 字符串函数

- sub 函数匹配记录中最大、最靠左边的子字符串的正则表达式，并用替换字符串替换这些字符串。如果没有指定目标字符串就默认使用整个记录。替换只发生在第一次匹配的时候。

格式:

```
sub (regular expression, substitution string);
sub (regular expression, substitution string, target string)
```

实例:

```
$ awk '{ sub(/test/, "mytest"); print }' testfile
$ awk '{ sub(/test/, "mytest", $1); print }' testfile
```

- gsub 函数作用如 sub，但它在整个文档中进行匹配。
- index 函数返回子字符串第一次被匹配的位置，偏移量从位置1开始。

格式:

```
index(string, substring);
```

实例:

```
$ awk '{ print index("test", "mytest") }' testfile
```

- length函数返回记录的字符数。

格式:

```
length( string );
length
```

实例:

```
$ awk '{ print length( "test" ) }' testfile
$ awk '{ print length }' testfile

第二个实例输出 testfile 文件中每条记录的字符数。
```

- substr 函数返回从位置1开始的子字符串，如果指定长度超过实际长度，就返回整个字符串。

格式:

```
substr( string, starting position );
substr( string, starting position, length of string );
```

实例:

```
$ awk '{ print substr( "hello world", 7, 11 ) }' testfile
```

- match 函数返回在字符串中正则表达式位置的索引，如果找不到指定的正则表达式则返回 0。match 函数会设置内建变量 RSTART 为字符串中子字符串的开始位置，RLENGTH 为到子字符串末尾的字符个数。substr 可利于这些变量来截取字符串。

格式:

```
match( string, regular expression );
```

实例:

```
$ awk '{start = match("this is a test", /[a-z]+$/); print start}' testfile
$ awk '{start = match("this is a test", /[a-z]+$/); print start, RSTART, RLENGTH}' testfile
```

- toupper 和 tolower 函数可用于字符串大小间的转换，该功能只在 gawk 中有效。

格式:

```
toupper( string );
tolower( string );
```

- split 函数可按给定的分隔符把字符串分割为一个数组。如果分隔符没提供，则按当前 FS 值进行分割

格式:

```
split( string, array, field separator );
split( string, array )
```

实例:

```
$ awk '{ split( "20:18:00", time, ":" ); print time[2] }' testfile
```

- 格式化输出. awk 提供了 printf() 和 sprintf()，等同于相应的 C 语言函数。printf() 会将格式化字符串打印到 stdout，而 sprintf() 则返回可以赋值给变量的格式化字符串。在 Linux 系统上，可以输入 "man 3 printf" 来查看 printf() 帮助页面。

格式:

```
sprintf(Format, Expr, Expr, . . . );
```

##  时间函数

- systime 函数返回从`1970年1月1日`开始到当前时间(不计闰年)的整秒数。

格式和实例:

```
systime();

$ echo "" | awk '{ now = systime(); print now }'
```

- strftime 函数使用 C 库中的 strftime 函数格式化时间。

格式:

```
strftime( [format specification][,timestamp] );
```

日期和时间格式说明符

格式  |  描述
----- | ----
%a | 星期几的缩写(Sun)
%A | 星期几的完整写法(Sunday)
%b | 月名的缩写(Oct)
%B | 月名的完整写法(October)
%c | 本地日期和时间
%d | 十进制日期
%D | 日期 08/20/99
%e | 日期，如果只有一位会补上一个空格
%H | 用十进制表示24小时格式的小时
%I | 用十进制表示12小时格式的小时
%j | 从1月1日起一年中的第几天
%m | 十进制表示的月份
%M | 十进制表示的分钟
%p | 12小时表示法(AM/PM)
%S | 十进制表示的秒
%U | 十进制表示的一年中的第几个星期(星期天作为一个星期的开始)
%w | 十进制表示的星期几(星期天是0)
%W | 十进制表示的一年中的第几个星期(星期一作为一个星期的开始)
%x | 重新设置本地日期(08/20/99)
%X | 重新设置本地时间(12：00：00)
%y | 两位数字表示的年(99)
%Y | 当前月份
%Z | 时区(PDT)
%% | 百分号(%)

## 数学函数

函数名称  |  返回值
--------- | ------
atan2(x,y) | y,x范围内的余切
cos(x) | 余弦函数
exp(x) | 求幂
int(x) | 取整，过程没有舍入
log(x) | 自然对数
rand() | 随机数
sin(x) | 正弦
sqrt(x) | 平方根
srand(x)  |  x是rand()函数的种子
rand() | 产生一个大于等于0而小于1的随机数

## 自定义函数

在awk中还可自定义函数，格式如下：

```
function name ( parameter, parameter, parameter, ... ) {
    statements
    return expression   # the return statement and expression are optional
}
```

## 调用 shell 命令 system 函数

格式:

```
system(cmd);
```

system() 的返回值是 cmd 的退出状态.如果要获得 cmd 的输出,就要和 `getline` 结合使用.

***getline 会增加 awk 编程复杂度,吃力不讨好,建议用别的可行方案代替,不到万不得已,别查 getline.***

*systime, strftime 在 gawk 下有效*

