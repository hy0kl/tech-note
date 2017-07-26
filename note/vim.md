# vim 光标移动

[FROM](http://www.ccvita.com/433.html)

```
基本的光标移动

h   左，或Backspace 或方向键。
j   下，或Enter 或+（要Shift 键），或方向键。
k   上，或方向键或-（不必Shift 键）。
l   右，或Space 或方向键。
Ctrl+f  即PageDown 翻页（Forward，向前、下翻页）。
Crtl+b  即PageUp 翻页（Backward，向后、上翻页)。

同样的，比如20h，就是光标向左移动20字符；20j，光标向下移动20字符；其他类似。
使用 hjkl
键的移动是为了使手不必离开打字区（键盘中央的部位），以加快打字的速度，如果各位不习惯，那就使用方向键吧！其实，一旦习惯了以后，对于编辑工作的效率
会有很大的帮助，而且有许多工作站的vi 只能使用hjkl 的移动方式，因此可能的话，尽量熟悉hjkl 的光标移动。

Backspace及Space的移动方式是到了行首或行尾时会折行，但方向键或hl 键的移动则在行首或行尾时您继续按也不会折行。转折换行的功能是Vim的扩充功能，elvis 无此功能。
jk 及使用方向键的上下移动光标会尽量保持在同一栏位。使用Enter，+，-的上下移动，光标会移至上（下）一行的第一个非空白字元处。

好像有点复杂，各位就暂时使用方向键来移动就简单明白了！等您爱上了Vim后再来讲究吧。

进阶的光标移动

0   是数目字0 而不是英文字母o。或是Home 键，移至行首，（含空白字元）。
^   移至行首第一个非空白字元，注意，要Shift 键。
$   移至行尾，或End 键。要 Shift 键。
G   移至档尾（全文最后一行的第一个非空白字元处）
gg  移至档首（全文第一行之第一个非空白字元处）。

在规则表示式（regular expression）中，^ 是匹配行首，$ 是匹配行尾。
gg 是Vim的扩充功能，在elvis 或原始vi 中可用1G 来移至档首（是数字1 不是英文字l ）。 G 之原意是goto，指移至指定数目行之行首，如不指定数目，则预设是最后一行。

w   移至次一个字（word）字首。当然是指英文单字。
W   同上，但会忽略一些标点符号。
e   移至后一个字字尾。
E   同上，但会忽略一些标点符号。
b   移至前一个字字首。
B   同上，但会忽略一些标点符号。
H   移至屏幕顶第一个非空白字元。
M   移至屏幕中间第一个非空白字元。
L   移至屏幕底第一个非空白字元。这和PageDown，PageUp 不一样，内文内容并未动，只是光标在动而已。
n|  移至第n 个字元(栏)处。注意，要用 Shift 键。 n 是从头起算的。
:n  移至第n 行行首。或 nG。

特殊的移动

)   移至下一个句子（sentence）首。
(   移至上一个句子（sentence）首。 sentence（句子）是以 . ! ? 为区格。
}   移至下一个段落（paragraph）首。
{   移至上一个段落（paragraph）首。 paragraph（段落）是以空白行为区格。
%   这是匹配{}，[]，() 用的，例如光标在{ 上只要按%，就会跑到相匹配的} 上。
```

# vim7 中文显示问题

[vim中文显示问题](http://moralistxp.blog.163.com/blog/static/1161103982009101111364351/)

安装了vim7.2，下载地址 [www.vim.org](http://www.vim.org)，但是在utf-8下输入中文却是乱码。
使用 :set 命令，发现vimrc中设置的字符集并未生效。
后发现是安装时 configuration 少写了参数；添加如下参数：

```
./configure --enable-multibyte
```

## 安装最新版vim

1. 选择最新版`https://github.com/vim/vim/releases`
  `wget https://github.com/vim/vim/archive/v版本号.tar.gz`
1. 解压,编译,安装

```
$ mv v版本号.tar.gz vim-v版本号.tar.gz
$ tar xf vim-v版本号.tar.gz
$ cd vim-版本号
$ ./configure --prefix=/usr/local --enable-multibyte
# 视情况和权限而定
$ ./configure --prefix=/home/work/local --enable-multibyte
# 没有权限就 sudo
$ make && make install
```

就可以正常显示输入的中文了。
附上vimrc的部分内容:

```
let &termencoding=&encoding
set encoding=utf-8
set fileencoding=utf-8
set fileencodings=utf-8,gb18030,utf-16,big5
set langmenu=C
```
注: 如果不指定  --enable-multibyte, set enc, set fileencoding 时发现没有这样的指令.

# 启动 vim 的一些选项

[see](http://www.cnblogs.com/jiqingwu/archive/2012/06/14/vim_notes.html)

```
vim -c cmd file: 在打开文件前，先执行指定的命令；
vim -r file: 恢复上次异常退出的文件；
vim -R file: 以只读的方式打开文件，但可以强制保存；
vim -M file: 以只读的方式打开文件，不可以强制保存；
vim -y num file: 将编辑窗口的大小设为num行；
vim + file: 从文件的末尾开始；
vim +num file: 从第num行开始；
vim +/string file: 打开file，并将光标停留在第一个找到的string上。
vim --remote file: 用已有的vim进程打开指定的文件。 如果你不想启用多个vim会话，这个很有用。但要注意， 如果你用vim，会寻找名叫VIM的服务器；如果你已经有一个gvim在运行了， 你可以用gvim --remote file在已有的gvim中打开文件。
```

# vim 将回车替换为特殊字符 + 回车

```
:%s/\n/;\r/g
```

## vim 中将空格替换成换行

```
%s/ \+/\r/g
```

***注: 查换时用 \n, 替换时用 \r, 这些什么神逻辑!***

# [vim 的匹配删除](http://luxiaok.blog.51cto.com/2177896/965465)

## 1. 删除含有"#"开头的行

```
:%g/^#/d
```

匹配删除含有特定字符的行就去掉“^”，也可以匹配结尾“$”

## 2. 删除空行

```
# 删除没有内容的空行
:%g/^$/d

# 删除包含有空格组成的空行
:g/^\s*$/d

# 删除以空格或\t开头到结尾的空行
:g/^[ |\t]*$/d
```

## 3. 删除不含该字符串的行

```
:% v/pattern/d
:% g!/pattern/d
```

## 4. 对每行只保留匹配内容而删除这一行中的其它内容

```
:%s/^.*\(pattern\).*$/\1/g
```

## 5. 删除包含特定字符串的行,每次删除前都提示

```
:%s/^.*pattern.*\n//c
```

删除有乱码`□`的行:

```
:g/\W*□\W$/d
```

## 6. 删除行末空白

```
:%s/\s\+$//g
```

# 在 CSS 中对属性进行排序

在可视模式下选择选中的行并输入 `:sort`，然后就可以对它们进行排序。

## 在 vim 中用密码保护文件

```
害怕 root 用户或者其他人偷窥你的个人文件么？尝试在vim中用密码保护，输入：

$ vim +X filename

或者，在退出 vim 之前使用 :X 命令来加密你的文件，vim 会提示你输入一个密码。

加密文件不需要加密，即停止对一个文件加密，可以把 'key' 选项设置为一个空字串：set key=
然后在命令行模式输入：wq（保存退出）
这样就可以把已加密文件去掉密码了。
```

# vim 中查找替换

```
:%s/abc/789/gc
```

每次遇到`abc`都会交互下以确认是否需要替换.

# 如何粘贴后自动整理代码,省去 = 对齐代码的重复操作?

```
]p
```

# vim中查找精确匹配或完全匹配

```
假设要查找的内容为abc, 要求精确匹配,则: /\<abc\> 即用 \<\> 将内容包起来,再如精确匹配www: /\<www\> 前匹配和后匹配.可以从上面的做法推导出来.
```

# vim特殊字符输入

```
# tab 的输入方法
CTRL + v + TAB

# windows 换行符
ctrl + v ctrl + m
```

