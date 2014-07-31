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

# ubuntu 设置静态 ip 地址

[参考](http://forum.ubuntu.org.cn/viewtopic.php?p=519187)

```
网络设置信息存储在/etc/network/interfaces文件中，默认dhcp的配置为：

# The primary network interface
auto eth0
iface eth0 inet dhcp

auto eth0
指明eth0在系统启动时自动加载
iface eth0 inet dhcp
指明eth0采用ipv4地址、由dhcp自动分配(inet表示ipv4地址，inet6表示ipv6地址)

注释掉#iface eth0 inet dhcp，修改为：

auto eth0
#iface eth0 inet dhcp
iface eth0 inet static
address 192.168.0.9
netmask 255.255.255.0
gateway 192.168.0.1

保存/退出

在静态ip下，还需要配置dns服务器。dns信息存储在/etc/resolv.conf中
由于之前由dhcp分配地址，所以resolv.conf中已经存储了dns信息，在没有改变网络环境时这一信息并不需要修改,内容如下(北京网通的ADSL):
nameserver 210.82.5.1
nameserver 202.106.46.151
全部修改完成后执行

sudo ifdown eth0 // 禁用网卡
sudo ifup eth0 // 启动网卡

使得设置生效。注意：此处一定要先ifdown再ifup；ifconfig命令也有up/down的参数，但似乎是用起来有问题，此处不要采用
```
