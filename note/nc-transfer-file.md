# 使用 nc 传输文件

```
1. 在远程服务器上
$ sudo /etc/init.d/iptables stop
$ nc -l 54321 > 传输文件目标名

2. 本地
$ nc 192.168.3.201 54321 < 待传输的文件

3. 传送文件夹
以传送 nginx 文件夹为例子.
tar 命令中加入 v 可以查看正在传送的文件

接收端:
$ nc -l 54321 | tar zxvf -

发送端:
$ tar zcf - nginx | nc 192.168.3.201 54321
```