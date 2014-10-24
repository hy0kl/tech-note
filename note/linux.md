# linux 下内存的统计

$ cat /proc/meminfo

# history 命令字统计游戏

```
% history | awk '{CMD[$2]++; count++;}END { for (a in CMD)\
print CMD[a] " " CMD[a]/count*100 "% " a;}' | grep -v "./" \
| column -c3 -s " " -t | sort -nr | nl |  head -n10
```

# Ubuntu 的桌面死机后重启桌面方法

[FROM](http://pppboy.blog.163.com/blog/static/30203796201173013345251/)

```
一、现状
大家都爱折腾一下Ubuntu的图形界面，调一下3D什么的，可能会由于你的过分折腾而死去。或者其它软件可能也会导致死去。

二、解决办法
从上到下依次优先

1.如果配置了ctrl+alt+backspace，先试这个，用它重启桌面环境

2.如果上面不行
在alt+ctrl+f1~F6中重启gdm服务：
    sudo /etc/init.d/gdm restart
或
    sudo /etc/init.d/gdm stop
    sudo /etc/init.d/gdm start

3. 在alt+ctrl+f1~F6中进入命令行Console运行：
ps -t tty7
此时可以发现一个Xorg的进程，记下他的PID。随后使用
kill 进程号

4.重启整个系统
alt+ctrl+f1~F6

三、出处
http://cppkey.com/
http://pppboy.blog.163.com/

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

