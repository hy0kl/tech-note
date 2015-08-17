# linux 下内存的统计

$ cat /proc/meminfo

# history 命令字统计游戏

```
% history | awk '{CMD[$2]++; count++;}END { for (a in CMD)\
print CMD[a] " " CMD[a]/count*100 "% " a;}' | grep -v "./" \
| column -c3 -s " " -t | sort -nr | nl |  head -n10
```

# 查看 linux 最大可以打开文件的数目

```
$ cat /proc/sys/fs/file-max
```

# linux shell 常见术语

```
IFS(Internal Field Seperator)
```

# crontab 以日期为日志名重定向的坑

% 在 crontab 里面是有特殊的意义的,需要使用 \ 进行转义

```
参考例子,每分钟产生一个随机数,写入指定的日志文件中:

*   *   *   *   *   head -200 /dev/urandom | cksum | cut -f1 -d" " 2>&1 >> /var/log/t.`/bin/date +'\%Y-\%m-\%d'`.log

*   *   *   *   *   head -200 /dev/urandom | cksum | cut -f1 -d" " 2>&1 >> /var/log/t.`/bin/date +\%Y-\%m-\%d`.log

ps: 实测通过.
$ uname -a
Linux wrok-dev 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

# curl 命令行中加入 UA

以下例子只列出百度的 http 头信息. user agent 用的是本机 Chrome.

$ curl --head  --verbose --user-agent 'User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.104 Safari/537.36' 'www.baidu.com'

```
-k, --insecure      Allow connections to SSL sites without certs (H)
    --interface INTERFACE  Specify network interface/address to use

-K, --config FILE   Specify which config file to read
    --connect-timeout SECONDS  Maximum time allowed for connection

-d, --data DATA     HTTP POST data (H)
     --data-ascii DATA  HTTP POST ASCII data (H)
     --data-binary DATA  HTTP POST binary data (H)
     --data-urlencode DATA  HTTP POST data url encoded (H)
     --delegation STRING GSS-API delegation permission
     --digest        Use HTTP Digest Authentication (H)
     --disable-eprt  Inhibit using EPRT or LPRT (F)
     --disable-epsv  Inhibit using EPSV (F)
     --dns-servers    DNS server addrs to use: 1.1.1.1;2.2.2.2
     --dns-interface  Interface to use for DNS requests
     --dns-ipv4-addr  IPv4 address to use for DNS requests, dot notation
     --dns-ipv6-addr  IPv6 address to use for DNS requests, dot notation

-D, --dump-header FILE  Write the headers to this file
    --egd-file FILE  EGD socket path for random data (SSL)
    --engine ENGINE  Crypto engine (SSL). "--engine list" for list

-i, --include       Include protocol headers in the output (H/F)

-v, --verbose       Make the operation more talkative
```

# linux下如何删除 '-' 开头的文件

[FROM](http://hi.baidu.com/zdz8207/item/85343a1855f47517e3f986b2)

```
不小心操作建立了一个 -ErrorMsg.txt 的文件

$ rm "-ErrorMsg.txt"
rm: invalid option -- E
Try `rm ./-ErrorMsg.txt' to remove the file `-ErrorMsg.txt'.
Try `rm --help' for more information.

$ rm -rf "-ErrorMsg.txt"
rm: invalid option -- E
Try `rm ./-ErrorMsg.txt' to remove the file `-ErrorMsg.txt'.
Try `rm --help' for more information.

使用rm 不能删除,根据提示用 rm ./-ErrorMsg.txt 可以.查看 rm 的 man 手册时有这么一行提示:
QUOTE:
To remove a file whose name starts with a ‘-’, for example ‘-foo’, use one of these commands:
    rm -- -foo
    rm ./-foo

这个指令是指删除当前目录 . 下的 -foo, 用 / 转义,如果要删除一个文件 -foo 不能用 rm 命令删除.
rm -foo, rm \-foo, rm "-foo", rm "\-foo" …… 都无法将此文件删除,只能通过 rm -- -foo 或者 rm ./-foo 的方式删除
同样此方法对于其它命令都是通用的
vi -- -c 将生成一个 -c 文件
ls -l -- -c 将显示 -c 文件
```

# linux下删除文件名乱码文件

[see](http://www.netingcn.com/linux-find-delete-file.html)

```
linux 下通过 rm 命令来删除文件，但是如果要删除文件名乱码的文件，就不能直接使用 rm 命令了，因为压根就无法输出文件名来。不过借助 find 命令可以实现对其删除。在 linux 下对于每个文件都一个对应的不变的 inode 号，使用 ls -li 可以查看到文件的 inode 号，同时 find 可以根据 inode 号来查找，另外 find 命令中可以执行其他的命令。删除的步骤如下：

通过 ls -li 获取要删除乱码文件名文件的 inode 号，比如得到的是 123456
执行删除
$ find ./ -inum 123456 -exec rm -rf {} \;
注意：“{}”后要空一格再加上“\;”。
```

# 一条命令统计实时并发数

```
将其中的 $4 换成日志中的时间字段即可
$ tail -f dev.access.log | awk 'BEGIN{OFS = "\t"; count = 0; iter_key = "check_key"}{count++; current = $4; if (iter_key != current) {print iter_key, count; count = 0; iter_key = current; }}'
```

# linux 下查看某个端口是否被占用

[FROM](http://my.oschina.net/u/193184/blog/146885)

```
查看端口占用情况的命令：lsof -i
$ sudo lsof -i
COMMAND     PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
smbd        733     root   30u  IPv6  10059      0t0  TCP *:microsoft-ds (LISTEN)
smbd        733     root   31u  IPv6  10060      0t0  TCP *:netbios-ssn (LISTEN)
smbd        733     root   32u  IPv4  10061      0t0  TCP *:microsoft-ds (LISTEN)
smbd        733     root   33u  IPv4  10062      0t0  TCP *:netbios-ssn (LISTEN)
nmbd        874     root   11u  IPv4   8874      0t0  UDP *:netbios-ns
...

查看某一端口的占用情况： lsof -i:端口号
$ sudo lsof -i:80
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx    2399  root   25u  IPv4  16050      0t0  TCP *:http (LISTEN)
nginx   30784 baihe   25u  IPv4  16050      0t0  TCP *:http (LISTEN)
nginx   30785 baihe   25u  IPv4  16050      0t0  TCP *:http (LISTEN)

```

# chmod 集锦

```
命令格式
$ chmod [-cfvR] [--help] [--version] mode file

必要参数:
-c 当发生改变时，报告处理信息
-f 错误信息不输出
-R 处理指定目录以及其子目录下的所有文件
-v 运行时显示详细处理信息

选择参数:
--reference=<目录或者文件> 设置成具有指定目录或者文件具有相同的权限
--version 显示版本信息
<权限范围>+<权限设置> 使权限范围内的目录或者文件具有指定的权限
<权限范围>-<权限设置> 删除权限范围的目录或者文件的指定权限
<权限范围>=<权限设置> 设置权限范围内的目录或者文件的权限为指定的值

权限范围：
u: 目录或者文件的当前的用户
g: 目录或者文件的当前的群组
o: 除了目录或者文件的当前用户或群组之外的用户或者群组
a: 所有的用户及群组

权限代号：
r: 读权限,用数字 4 表示
w: 写权限,用数字 2 表示
x: 执行权限,用数字 1 表示
-: 删除权限,用数字 0 表示

set位及粘滞位
权限值组合中的第4位数
对应关系: suid=4, sgid=2, 粘滞位=1
使用字母表示时对应关系: s=set位, t=粘滞位
set位含义: 设置 set 位以后,其他用户执行该文件时也会拥有设置该 set 位用户的对应身份和权限,一般设置在文件上
粘滞位含义: 设置粘滞位以后,让其他用户无法删除别人的文件,一般设置在目录上
```

# 如何判断自己的操作系统是32位还是64位?

```
方法一:
$ uname -m

方法二:
$ getconf LONG_BIT

方法三:
$ arch
```

# 取昨天的时间串

```
$ date +"%Y%m%d" -d "yesterday"
```

# sendemail 使用 163 smtp 代理发邮件报错

[官方](https://bugs.launchpad.net/ubuntu/+source/sendemail/+bug/1072299)
[From](http://raspberrypi.stackexchange.com/questions/2118/sendemail-failure)

```
Linux:
invalid SSL_version specified at /usr/share/perl5/IO/Socket/SSL.pm line 368.

MaxOS:
invalid SSL_version specified at /System/Library/Perl/Extras/5.18/IO/Socket/SSL.pm line 368.

$ sudo vim /usr/share/perl5/IO/Socket/SSL.pm

找到
m{^(!?)(?:(SSL(?:v2|v3|v23|v2/3))|(TLSv1(?:_?[12])?))$}i
替换为
m{^(!?)(?:(SSL(?:v2|v3|v23|v2/3))|(TLSv1[12]?))}i
```

# linux 终端不能输入中文解决方法

[参考](http://blog.sina.com.cn/s/blog_5c4dd3330100cpmm.html)

在用户目录下的 ~/.inputrc 文件（如果没有，则新建一个）添加：

```
set meta-flag on
set convert-meta off
set input-meta on
set output-meta on
```

如果还是不能输入中文，再试试在 /etc/profile 文件里添加：

```
LANG="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.eucCN"
export LANG LC_MESSAGES
```

# 如何不解压tar.gz文件查看其中的文件大小

[参考](http://www.codelast.com/?p=950)保留链接,小网站,有可能随时消失

```
$ tar tvf file.tar.gz

# 只显示文件大小和文件名
$ tar tvf file.tar.gz | awk '{print $3, $6}'

# 以 KB/MB/GB 来显示文件大小
$ tar tvf file.tar.gz | awk '{print $3 / 1024 / 1024 / 1024, $6}'
```

# diff

## 比较两个目录中文件差异

```
$ diff -u -r dir1 dir2
```

## 部分参数说明

[see](http://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html)

由于历史原因, diff 有三种格式

* 正常格式(normal diff)
* -c 上下文格式(context diff)
* -u 合并格式(unified diff)

## 其他常用参数

```
-t, --expand-tabs               expand tabs to spaces in output
-r, --recursive                 recursively compare any subdirectories found
    --no-dereference            don't follow symbolic links

-i, --ignore-case               ignore case differences in file contents
-E, --ignore-tab-expansion      ignore changes due to tab expansion
-Z, --ignore-trailing-space     ignore white space at line end
-b, --ignore-space-change       ignore changes in the amount of white space
-w, --ignore-all-space          ignore all white space
-B, --ignore-blank-lines        ignore changes where lines are all blank
-I, --ignore-matching-lines=RE  ignore changes where all lines match RE
```

# svn 报错

```
svn: warning: cannot set LC_CTYPE locale
svn: warning: environment variable LC_CTYPE is UTF-8
svn: warning: please check that your locale name is correct
```

[see](http://stackoverflow.com/questions/11300633/svn-cannot-set-lc-ctype-locale)

```
export LC_ALL=C
```

# git 添加过 key, push 时要求用户名和密码验证

```shell
$ uname -a
Linux work-dev 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

$ git push origin master
git-remote-https: /home/work/local/lib/libcurl.so.4: no version information available (required by git-remote-https)

$ sudo apt-get install libcurl4-openssl-dev
```

原因是用户自己的编译的 libcurl 没有 SSL support.

***openssl 能用系统提供的还是尽量用系统的吧,这个组件太变态了.***

# 查看 gateway 等网络情况

```shell
$ netstat -rn
```

# 个性化登陆欢迎词

先安装软件

```shell
$ sudo apt-get install fortune-zh cowsay
```

配置并使其生效

```shell
$ sudo vim /etc/profile

cowsay_file=`echo "" | awk 'BEGIN{ srand(); } { animal = "apt beavis.zen bong bud-frogs bunny calvin cheese cock cower daemon default dragon dragon-and-cow duck elephant elephant-in-snake eyes flaming-sheep ghostbusters gnu head-in hellokitty kiss kitty koala kosh luke-koala mech-and-cow meow milk moofasa moose mutilated pony pony-smaller ren sheep skeleton snowman sodomized-sheep stegosaurus stimpy suse three-eyes turkey turtle tux unipony unipony-smaller vader vader-koala www"; value = int(rand() * 1000); count = split(animal, animal_cntr, " "); print animal_cntr[value % count + 1];}'`
fortune | cowsay -f $cowsay_file
```

又找到一个更牛的方法[see](https://wiki.archlinux.org/index.php/Fortune):

```shell
$ fortune | cowthink -f $(find /usr/share/cowsay/cows -type f | shuf -n 1)
```

改良的方法:

```shell
cowsay_file=$(find /usr/share/cowsay/cows -type f | awk 'BEGIN{ i = 1; srand(); } {cntr[i] = $0; i++} END{ value = int(rand() * 1000); print cntr[value % (i - 1) + 1];}')
fortune | cowsay -f $cowsay_file
```

# 实用技巧收集

[from](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=205588580&idx=2&sn=eb0e399108cc4e79001e4d711ee4bc2d&key=b2574200810f04e8531657681efbaff73a3bc1c5683e0b9d5680e24ae90af6ee6f0ccc6522cc78633bce7c580afdc541&ascene=0&uin=MTQzNjU2MzQ2MQ%3D%3D&devicetype=iMac+MacBookPro11%2C1+OSX+OSX+10.10.2+build(14C1514)&version=11020012&pass_ticket=b%2Bt0c0kBZzWRHCg5%2BTjB5iYp2%2FKuFarbLymQkeDM%2BZtXXwE64E%2FqOoq9SMNpLF%2Fi)

## 删除一个大文件

```
$ /path/to/file.log
# 或使用如下格式
$ : > /path/to/file.log

# 然后删除它
$ rm /path/to/file.log
```

## 在vim中用密码保护文件

```
害怕 root 用户或者其他人偷窥你的个人文件么？尝试在vim中用密码保护，输入：

vim +X filename

或者，在退出 vim 之前使用 :X 命令来加密你的文件，vim 会提示你输入一个密码。

加密文件不需要加密，即停止对一个文件加密，可以把 'key' 选项设置为一个空字串：set key=
然后在命令行模式输入：wq（保存退出）
这样就可以把已加密文件去掉密码了。
```

## 清除屏幕上的乱码

```
$ reset
```

## 如何删除意外在当前文件夹下解压的文件？

```
意外在/var/www/html/而不是/home/projects/www/current下解压了一个tarball。它搞乱了/var/www/html下的文件，你甚至不知道哪些是误解压出来的。最简单修复这个问题的方法是：

$ cd /var/www/html/
$ /bin/rm -f "$(tar ztf /path/to/file.tar.gz)"
```

## 在 less 浏览时编辑文件

```
要编辑一个正在用 less 浏览的文件，可以按下 v。你就可以用变量 $EDITOR 所指定的编辑器来编辑了：

$ less *.c
$ less foo.html
## 按下v键来编辑文件 ##
## 退出编辑器后，你可以继续用less浏览了 ##
```

# kill 的信号量

```
Mac 下
These signals are defined in the file <signal.h>:

     No    Name         Default Action       Description
     1     SIGHUP       terminate process    terminal line hangup
     2     SIGINT       terminate process    interrupt program
     3     SIGQUIT      create core image    quit program
     4     SIGILL       create core image    illegal instruction
     5     SIGTRAP      create core image    trace trap
     6     SIGABRT      create core image    abort program (formerly SIGIOT)
     7     SIGEMT       create core image    emulate instruction executed
     8     SIGFPE       create core image    floating-point exception
     9     SIGKILL      terminate process    kill program
     10    SIGBUS       create core image    bus error
     11    SIGSEGV      create core image    segmentation violation
     12    SIGSYS       create core image    non-existent system call invoked
     13    SIGPIPE      terminate process    write on a pipe with no reader
     14    SIGALRM      terminate process    real-time timer expired
     15    SIGTERM      terminate process    software termination signal
     16    SIGURG       discard signal       urgent condition present on socket
     17    SIGSTOP      stop process         stop (cannot be caught or ignored)
     18    SIGTSTP      stop process         stop signal generated from keyboard
     19    SIGCONT      discard signal       continue after stop
     20    SIGCHLD      discard signal       child status has changed
     21    SIGTTIN      stop process         background read attempted from control terminal
     22    SIGTTOU      stop process         background write attempted to control terminal
     23    SIGIO        discard signal       I/O is possible on a descriptor (see fcntl(2))
     24    SIGXCPU      terminate process    cpu time limit exceeded (see setrlimit(2))
     25    SIGXFSZ      terminate process    file size limit exceeded (see setrlimit(2))
     26    SIGVTALRM    terminate process    virtual time alarm (see setitimer(2))
     27    SIGPROF      terminate process    profiling timer alarm (see setitimer(2))
     28    SIGWINCH     discard signal       Window size change
     29    SIGINFO      discard signal       status request from keyboard
     30    SIGUSR1      terminate process    User defined signal 1
     31    SIGUSR2      terminate process    User defined signal 2

不同系统还存在差别.

$ uname -a
Darwin MacPro.local 14.1.0 Darwin Kernel Version 14.1.0: Thu Feb 26 19:26:47 PST 2015; root:xnu-2782.10.73~1/RELEASE_X86_64 x86_64
$ kill -l
 1) SIGHUP   2) SIGINT   3) SIGQUIT  4) SIGILL
 5) SIGTRAP  6) SIGABRT  7) SIGEMT   8) SIGFPE
 9) SIGKILL 10) SIGBUS  11) SIGSEGV 12) SIGSYS
13) SIGPIPE 14) SIGALRM 15) SIGTERM 16) SIGURG
17) SIGSTOP 18) SIGTSTP 19) SIGCONT 20) SIGCHLD
21) SIGTTIN 22) SIGTTOU 23) SIGIO   24) SIGXCPU
25) SIGXFSZ 26) SIGVTALRM   27) SIGPROF 28) SIGWINCH
29) SIGINFO 30) SIGUSR1 31) SIGUSR2

$ uname -a
Linux work-dev 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
$ kill -l
 1) SIGHUP   2) SIGINT   3) SIGQUIT  4) SIGILL   5) SIGTRAP
 6) SIGABRT  7) SIGBUS   8) SIGFPE   9) SIGKILL 10) SIGUSR1
11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG  24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF 28) SIGWINCH    29) SIGIO   30) SIGPWR
31) SIGSYS  34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

# shell 中条件判断

[see](http://blog.csdn.net/ithomer/article/details/5904632)

```
[ -a FILE ]  如果 FILE 存在则为真。
[ -b FILE ]  如果 FILE 存在且是一个块特殊文件则为真。
[ -c FILE ]  如果 FILE 存在且是一个字特殊文件则为真。
[ -d FILE ]  如果 FILE 存在且是一个目录则为真。
[ -e FILE ]  如果 FILE 存在则为真。
[ -f FILE ]  如果 FILE 存在且是一个普通文件则为真。
[ -g FILE ] 如果 FILE 存在且已经设置了SGID则为真。 [ -h FILE ]  如果 FILE 存在且是一个符号连接则为真。
[ -k FILE ]  如果 FILE 存在且已经设置了粘制位则为真。
[ -p FILE ]  如果 FILE 存在且是一个名字管道(F如果O)则为真。
[ -r FILE ]  如果 FILE 存在且是可读的则为真。
[ -s FILE ]  如果 FILE 存在且大小不为0则为真。
[ -t FD ]  如果文件描述符 FD 打开且指向一个终端则为真。
[ -u FILE ]  如果 FILE 存在且设置了SUID (set user ID)则为真。
[ -w FILE ]  如果 FILE 如果 FILE 存在且是可写的则为真。
[ -x FILE ]  如果 FILE 存在且是可执行的则为真。
[ -O FILE ]  如果 FILE 存在且属有效用户ID则为真。
[ -G FILE ]  如果 FILE 存在且属有效用户组则为真。
[ -L FILE ]  如果 FILE 存在且是一个符号连接则为真。
[ -N FILE ]  如果 FILE 存在 and has been mod如果ied since it was last read则为真。
[ -S FILE ]  如果 FILE 存在且是一个套接字则为真。
[ FILE1 -nt FILE2 ]  如果 FILE1 has been changed more recently than FILE2, or 如果 FILE1 exists and FILE2 does not则为真。
[ FILE1 -ot FILE2 ]  如果 FILE1 比 FILE2 要老, 或者 FILE2 存在且 FILE1 不存在则为真。
[ FILE1 -ef FILE2 ]  如果 FILE1 和 FILE2 指向相同的设备和节点号则为真。
[ -o OPTIONNAME ]  如果 shell选项 “OPTIONNAME” 开启则为真。
[ -z STRING ]  “STRING” 的长度为零则为真。
[ -n STRING ] or [ STRING ]  “STRING” 的长度为非零 non-zero则为真。
[ STRING1 == STRING2 ]  如果2个字符串相同。 “=” may be used instead of “==” for strict POSIX compliance则为真。
[ STRING1 != STRING2 ]  如果字符串不相等则为真。
[ STRING1 < STRING2 ]  如果 “STRING1” sorts before “STRING2” lexicographically in the current locale则为真。
[ STRING1 > STRING2 ]  如果 “STRING1” sorts after “STRING2” lexicographically in the current locale则为真。
[ ARG1 OP ARG2 ] “OP” is one of -eq, -ne, -lt, -le, -gt or -ge. These arithmetic binary operators return true if “ARG1” is equal to, not equal to, less than, less than or equal to, greater than, or greater than or equal to “ARG2”, respectively. “ARG1” and “ARG2” are integers.
```

# 终端中文乱码

```
$ sudo vi /etc/profile
或
$ vi ~/.bashrc

# set encode
export LANG=C
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

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

# CentOS 查看内核版本，位数，版本号

总是查,索性记录一下.[see](http://blog.csdn.net/painsonline/article/details/7668824)

```
1)[root@localhost ~]# cat /proc/version
Linux version 2.6.18-194.el5 (mockbuild@builder10.centos.org) (gcc version 4.1.2 20080704 (Red Hat 4.1.2-48)) #1 SMP Fri Apr 2 14:58:14 EDT 2010

2)
[root@localhost ~]# uname -a
Linux localhost.localdomain 2.6.18-194.el5 #1 SMP Fri Apr 2 14:58:14 EDT 2010 x86_64 x86_64 x86_64 GNU/Linux

3)
[root@localhost ~]# uname -r
2.6.18-194.el5

2. 查看linux版本：
1) 列出所有版本信息,
[root@localhost ~]# lsb_release -a
LSB Version:    :core-3.1-amd64:core-3.1-ia32:core-3.1-noarch:graphics-3.1-amd64:graphics-3.1-ia32:graphics-3.1-noarch
Distributor ID: CentOS
Description:    CentOS release 5.5 (Final)
Release:        5.5
Codename:       Final
注:这个命令适用于所有的linux，包括Redhat、SuSE、Debian等发行版。

2) 执行cat /etc/issue,例如如下:
[root@localhost ~]# cat /etc/issue
CentOS release 5.5 (Final)
Kernel r on an m

3) 执行cat /etc/redhat-release ,例如如下:
[root@localhost ~]# cat /etc/redhat-release
CentOS release 5.5 (Final)

查看系统是64位还是32位:

1、getconf LONG_BIT or getconf WORD_BIT
[root@localhost ~]# getconf LONG_BIT
64

2、file /bin/ls
[root@localhost ~]# file /bin/ls
/bin/ls: ELF 64-bit LSB executable, AMD x86-64, version 1 (SYSV), for GNU/Linux 2.6.9, dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped

3、lsb_release  -a
[root@localhost ~]# lsb_release -a
LSB Version:    :core-3.1-amd64:core-3.1-ia32:core-3.1-noarch:graphics-3.1-amd64:graphics-3.1-ia32:graphics-3.1-noarch
Distributor ID: CentOS
Description:    CentOS release 5.5 (Final)
Release:        5.5
Codename:       Final
在linux中我们要操作任何东西都需要使用命令模式来操作了，所以如果想精通linux服务器的朋友可以多看看这方面的教程了，像我们这里查获系统版本都使用了几行命令了哦。
```

# crontab 执行不发邮件

```
$ crontab -e

+ MAILTO=""
```

# 查看服务监听的端口

```
$ netstat -nlp # linux 显示监听,进程
$ lsof -ni | grep LISTEN # mac 下类似上条命令效果
$ lsof -i:9000 # 查看9000端口的占用者
```

# 诡异的 date

```
$ date -d@1271397015 # debian Linux GNU/Linux
$ date "+%Y-%m-%d %H:%M" -d@1433327259
$ date -r 1271397015 # Mac OS/BSD
$ date -r 1433327259 "+%Y-%m-%d %H:%M"
```

# 命令行参数自动补全

```
$ vim ~/.bashrc

加入以下配置

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```

