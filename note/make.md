# note

```
VPATH = src:../headers # make 在当前目录找不到依赖文件,会去 VPATH 目录中去找,多个目录之间用 : 隔开
.PHONY : all install clean # 伪目标
%: 匹配零或若干字符
$< 表示所有的依赖目标集
$@ 表示目标集
用 @ 字符在命令行前,此命令将不被 make 显示出来
在 Makefile 的命令行前加一个减号 - (在 Tab 键之后),标记为不管命令出不出错都认为是成功的.
给 make 加上“-i”或是“--ignore-errors”参数,那么,Makefile 中所有命令都会忽略错误.
编译器 -M 参数,自动找寻源文件中包含的头文件,并 生成一个依赖关系.推荐 -MM, 防止把标准库头文件也包含进来.
export 传递变量,什么参数都不带,表示传递所有变量.
SHELL, MAKEFLAGS 变量总是要传递到下层 Makefile 中的.
make 命令中的有几个参数并不往下传递,它们是 -C, -f, -h, -o 和-W.
-w 或是 --print-directory 会在 make 的过程中输出一些信息.
-C 参数来指定 make 下层 Makefile 时, -w 会被自动打开的。如果参数中 有 -s(--slient)或是 --no-print-directory,那么 -w 总是失效的。
```

# 变量

```
为了防止变量替换出现递归,可以使用 := 来定义变量,如: var := value
?= 操作符表示如果变量定义过,则用原值,如果没有定义过,则使用新给定的值.
+= 操作符给变量追加值.
如果 make 指定了 -e 参数,那么,系统环境变量将覆盖 Makefile 中定义的变量.
```

## 变量替换

```
格式是“$(var:a=b)”或是“${var:a=b}”,其意思是,把变量“var”中所有以“a”字串“结尾”的“a”替换成“b”字串。这里的“结尾”意思是“空格”或是“结束符”。
另外一种变量替换的技术是以“静态模式”.
```

# 条件表达式

关键词 ifeq、else 和 endif。ifeq 的意思表示条件语句 的开始,并指定一个条件表达式,表达式包含两个参数,以逗号分隔,表达式以圆括 号括起。else 表示条件表达式为假的情况。endif 表示一个条件语句的结束,任何一个条 件表达式都应该以 endif 结束。

```
<conditional-directive>
    <text-if-true>
endif

<conditional-directive>
    <text-if-true>
else
    <text-if-false>
endif

<conditional-directive>表示条件关键字,有 ifeq, ifneq, ifdef, ifndef.
```

# 函数调用

```
$(<function> <arguments>)

或

${<function> <arguments>}

<arguments>是函数的参数,参数 间以逗号“,”分隔,而函数名和参数之间以“空格”分隔。
```

# 字符串处理函数

```
$(subst <from>,<to>,<text>)
名称:字符串替换函数——subst。
功能:把字串<text>中的<from>字符串替换成<to>。
返回:函数返回被替换过后的字符串。

$(patsubst <pattern>,<replacement>,<text>)
名称:模式字符串替换函数——patsubst。
功能:查找<text>中的单词(单词以“空格”、“Tab”或“回车”“换行”分隔)是否符合模式<pattern>,如果匹配的话,则以<replacement>替换。这里,<pattern>可以包括通配符“%”,表示任意长度的字串。如果<replacement>中也包含“%”,那么, <replacement>中的这个“%”将是<pattern>中的那个“%”所代表的字串。(可以用“\” 来转义,以“\%”来表示真实含义的“%”字符)
返回:函数返回被替换过后的字符串。

$(strip <string>)
名称:去空格函数——strip。
功能:去掉<string>字串中开头和结尾的空字符。
返回:返回被去掉空格的字符串值。

$(findstring <find>,<in>)
名称:查找字符串函数——findstring。
功能:在字串<in>中查找<find>字串。
返回:如果找到,那么返回<find>,否则返回空字符串。

$(filter <pattern...>,<text>)
名称:过滤函数——filter。
功能:以<pattern>模式过滤<text>字符串中的单词,保留符合模式<pattern>的单词。可以有多个模式。
返回:返回符合模式<pattern>的字串。

$(filter-out <pattern...>,<text>)
名称:反过滤函数——filter-out。
功能:以<pattern>模式过滤<text>字符串中的单词,去除符合模式<pattern>的单词。可以有多个模式。
返回:返回不符合模式<pattern>的字串。

$(sort <list>)
名称:排序函数——sort。
功能:给字符串<list>中的单词排序(升序)。
返回:返回排序后的字符串。
备注:sort 函数会去掉<list>中相同的单词。

$(word <n>,<text>)
名称:取单词函数——word。
功能:取字符串<text>中第<n>个单词。(从一开始)
返回:返回字符串<text>中第<n>个单词。如果<n>比<text>中的单词数要大,那么返回 空字符串。

$(wordlist <s>,<e>,<text>)
名称:取单词串函数——wordlist。
功能:从字符串<text>中取从<s>开始到<e>的单词串。<s>和<e>是一个数字。
返回:返回字符串<text>中从<s>到<e>的单词字串。如果<s>比<text>中的单词数要大, 那么返回空字符串。如果<e>大于<text>的单词数,那么返回从<s>开始,到<text>结束的单词串。

$(words <text>)
名称:单词个数统计函数——words。
功能:统计<text>中字符串中的单词个数。
返回:返回<text>中的单词数。
备注:如果我们要取<text>中最后的一个单词,我们可以这样:$(word $(words <text> ),<text>)。

$(firstword <text>)
名称:首单词函数——firstword。
功能:取字符串<text>中的第一个单词。
返回:返回字符串<text>的第一个单词。
备注:这个函数可以用 word 函数来实现:$(word 1,<text>)。

shell 函数
contents := $(shell cat test.c)

控制函数
$(error <text ...>)

警告
$(warning <text ...>)
```

# make 的退出码

```
0 —— 表示成功执行。
1 —— 如果 make 运行时出现任何错误,其返回 1。
2 —— 如果你使用了 make 的“-q”选项,并且 make 使得一些目标不需要更新,那么返 回2。
```

# 例子

```
%.o: %.c
    $(CC) $(CCFLAGS) -c $< -o $@
```

```
# 嵌套执行 make
subsystem:
    cd subdir && $(MAKE)

# 等价于
subsystem:
    $(MAKE) -C subdir

# 阻止往下层传递参数
subsystem:
    cd subdir && $(MAKE) MAKEFLAGS=

# 定义一个变量,其值是一个空格
nullstring :=
space := $(nullstring) # end of the line

# 静态模式变量替换
foo := a.o b.o c.o
bar := $(foo:%.o=%.c)
# 让$(bar)变量的值为“a.c b.c c.c”
```
