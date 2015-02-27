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
``

[see](http://stackoverflow.com/questions/11300633/svn-cannot-set-lc-ctype-locale)

```
export LC_ALL=C
```

# git 添加过 key, push 时要求用户名和密码验证

```
$ uname -a
Linux work-dev 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

$ git push origin master
git-remote-https: /home/work/local/lib/libcurl.so.4: no version information available (required by git-remote-https)
```

```
sudo apt-get install libcurl4-openssl-dev
```

原因是用户自己的编译的 libcurl 没有 SSL support.

***openssl 能用系统提供的还是尽量用系统的吧,这个组件太变态了.***

# 查看 gateway 等网络情况

```
$ netstat -rn
```

# 个性化登陆欢迎词

```
$ sudo vim /etc/profile

+

cowsay_file=`echo "" | awk 'BEGIN{ srand(); } { animal = "apt beavis.zen bong bud-frogs bunny calvin cheese cock cower daemon default dragon dragon-and-cow duck elephant elephant-in-snake eyes flaming-sheep ghostbusters gnu head-in hellokitty kiss kitty koala kosh luke-koala mech-and-cow meow milk moofasa moose mutilated pony pony-smaller ren sheep skeleton snowman sodomized-sheep stegosaurus stimpy suse three-eyes turkey turtle tux unipony unipony-smaller vader vader-koala www"; value = int(rand() * 1000); count = split(animal, animal_cntr, " "); print animal_cntr[value % count + 1];}'`
echo $f;            
fortune | cowsay -f $cowsay_file
```

