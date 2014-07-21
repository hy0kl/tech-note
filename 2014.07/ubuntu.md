# ubuntu 关闭图形界面的命令是什么

[参考](http://segmentfault.com/q/1010000000369635)

```
$ sudo /etc/init.d/lightdm stop
```

# 如何查看 ubuntu 的内核版本和发行版本号

[参考](http://blog.csdn.net/debug_cpp/article/details/2687067)

```
$ sudo lsb_release -a
$ uname -a
```

# ubuntu 服务器 ssh 登陆后无法输入中文

```
现象:
$ locale
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=zh_CN.UTF-8
LANGUAGE=en_US:en
LC_CTYPE=UTF-8
LC_NUMERIC=en_US.UTF-8
LC_TIME=en_US.UTF-8
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY=en_US.UTF-8
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER=en_US.UTF-8
LC_NAME=en_US.UTF-8
LC_ADDRESS=en_US.UTF-8
LC_TELEPHONE=en_US.UTF-8
LC_MEASUREMENT=en_US.UTF-8
LC_IDENTIFICATION=en_US.UTF-8
LC_ALL=
```

```
解决方法:
$ sudo vim /etc/environment

# add conf
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8

重新登陆即可
```
