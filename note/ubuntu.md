# ubuntu 关闭图形界面的命令是什么

[参考](http://segmentfault.com/q/1010000000369635)

```
$ sudo /etc/init.d/lightdm stop
```

# 如何查看 ubuntu 的内核版本和发行版本号

[参考](http://blog.csdn.net/debug_cpp/article/details/2687067)

```
$ cat /etc/issue
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

最新的换了[20151105]

/etc/init.d/lightdm

```

# 重启后 ping 不通外网

```
主要原因是 dns 配置丢失,解决方法有两种:

1. 临时有效
$ sudo vim /etc/resolv.conf
加入以下内容
nameserver 8.8.8.8
nameserver 114.114.114.114

2. 永久有效
$ sudo vim /etc/resolvconf/resolv.conf.d/base
加入以下内容
nameserver 8.8.8.8
nameserver 114.114.114.114

执行以下命令,强制生效
$ sudo resolvconf -u
```

# 添加用户到已存在的组

[参考](http://linux.cn/thread-11790-1-1.html)

```
$ sudo adduser 用户名 组名
$ id 用户名
```

# ubuntu 下干干净净的卸载软件

[see](http://oss.org.cn/html/47/n-67447.html)

```
1. apt 方式
  1). 一般卸载 sudo apt-get remove softname1 softname2 ...
  2). 清除式卸载 sudo apt-get --purge remove softname1 softname2 ... # 同时清除配置
  3). 同上 sudo apt-get purge sofname1 softname ...

2. dpkg 方式
  1). 移除式卸载 sudo dpkg -r pkg1 pkg2 ...
  2). 清除式卸载 sudo dpkg -P pkg1 pkg2 ...
```

# setlocale: LC_MESSAGES: cannot change locale (zh_CN.eucCN): No such file or directory

[see](http://wiki.ubuntu.org.cn/%E4%BF%AE%E6%94%B9locale)

# perl 报错

```
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
    LANGUAGE = "en_US:en",
    LC_ALL = (unset),
    LC_CTYPE = "UTF-8",
    LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
```

解决方法

[参考](http://stackoverflow.com/questions/2499794/how-can-i-fix-a-locale-warning-from-perl)

```
# vi /etc/profile

export LANG=C
export LC_ALL=en_US.UTF-8
```

# sudo 不需要密码

```
# vim /etc/sudoers

%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

# 如何在vim保存时获得sudo权限

[see](http://segmentfault.com/q/1010000000151086)

```
:w !sudo tee %
```

# libstdc++.so.6: cannot open shared object file: No such file or directory

```
$ sudo apt-get install lib32stdc++6
```

# while loading shared libraries: libz.so.1: cannot open shared object file: No such file or directory
[see](http://askubuntu.com/questions/396473/error-with-libz-so-1-on-android-studio)

```
$ sudo apt-get install lib32z1 lib32ncurses5
```

# /usr/bin/ld: skipping incompatible /usr/lib/gcc/x86_64-linux-gnu/4.9/libgcc.a when searching for -lgcc

[see](http://askubuntu.com/questions/85978/building-a-32-bit-app-in-64-bit-ubuntu)

```
$ sudo apt-get install gcc-multilib
```

# 交叉编译报 /usr/include/bits/mathinline.h error: impossible constraint in 'asm'

```
$ export LIBRARY_PATH=""
$ export C_INCLUDE_PATH=""
$ export CPLUS_INCLUDE_PATH=""
```

# arm-linux-gnueabihf-gcc: Command not found

[see](http://stackoverflow.com/questions/14180185/gcc-arm-linux-gnueabi-command-not-found)

```
$ sudo apt-get install ia32-libs

if you are on 64 bit os then you need to install this additional libraries.
$ sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0

貌似没啥用
``

[abi安装](http://packages.ubuntu.com/precise/devel/gcc-arm-linux-gnueabi)
[eabi安装](http://packages.ubuntu.com/zh-cn/precise/devel/gcc-arm-linux-gnueabihf)

# build.xml:396: SDK Platform Tools component is missing. Please install it with the SDK Manager (tools/android)

[see](http://stackoverflow.com/questions/16631693/android-build-failing-with-build-xml479-sdk-does-not-have-any-build-tools-inst)

```
$ android update sdk -u
```

# import _tkinter # If this fails your Python may not be configured for Tk

```
$ sudo apt-get install python3-tk # 然后重新编译python3
$ sudo apt-get install python-tk  # python2 的报错解决
```

# virtualbox中的Ubuntu添加磁盘

[see](https://blog.csdn.net/u011774239/article/details/50460076)


VitrualBox是不允许更改重置硬盘大小的，所以当硬盘不足时，只能添加新硬盘。

步骤如下：

1. 关闭Ubuntu系统，打开VistualBox，"设置"->"存储"->“添加虚拟硬盘”
2. 启动Ubuntu系统，操作命令如下：

```
#1 sudo fdisk -l // 查看现有系统磁盘空间
----------------------------------------------------------------------------
Disk /dev/sda: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Disk identifier: 0x000af383

Device Boot Start End Blocks Id System
/dev/sda1 * 1 1244 9992398+ 83 Linux
/dev/sda2 1245 1305 489982+ 5 Extended
/dev/sda5 1245 1305 489951 82 Linux swap / Solaris

Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Disk identifier: 0x00000000

Disk /dev/sdb doesn't contain a valid partition table
----------------------------------------------------------------------------

以上信息可以看到新增加的磁盘空间 /dev/sdb ，这里我们需要给新的磁盘空间分区。
#2 fdisk /dev/sdb
#3 Command (m for help): m // 键入m，可看到帮助信息
打印结果如下：
----------------------------------------------------------------------------
Command action
a toggle a bootable flag
b edit bsd disklabel
c toggle the dos compatibility flag
d delete a partition
l list known partition types
m print this menu
n add a new partition
o create a new empty DOS partition table
p print the partition table
q quit without saving changes
s create a new empty Sun disklabel
t change a partition's system id
u change display/entry units
v verify the partition table
w write table to disk and exit
x extra functionality (experts only)
----------------------------------------------------------------------------
#4 Command (m for help): n
打印结果如下：
----------------------------------------------------------------------------
Command action
e extended
p primary partition (1-4)
----------------------------------------------------------------------------

键入：p，选择添加主分区；
键入：1，选择主分区编号为1， 这样创建后的主分区为sdb1；

#5 First Cylinder(1-1014,default 1): 1 // 第一个主分区起始的磁盘块数
#6 Last cylindet or +siza or +sizeM or +sizeK: +1024MB // 可以是以MB为单位的数字或者以磁盘块数，这里我们输入+1024MB表示分区大小为1G。

这样我们就创建完一个分区，如果要创建更多分区可以照上面的步骤继续创建。
最后，键入：w，保存所有并退出，完成新磁盘的分区。
```

4. 格式化磁盘分区

```
#7 sudo mkfs -t ext4 /dev/sdb1 // 用ext4格式对 /dev/sdb1 进行格式化
```

5. 挂载分区

```
#8 sudo mkdir /data // 创建新的挂载点
#9 sudo mount /dev/sdb1 /data // 将新磁盘分区挂载到 /data 目录下
#10 sudo df // 查看挂载结果
```
6. 开机自动挂载

```
#11 vi /etc/fstab // 修改 /etc/fstab 文件
在 /etc/fstab 文件中，添加如下内容：
/dev/sdb1 /data ext4 defaults 1 2
```

亲测有效

# Ubuntu 18.04.1 LTS

## 初始化

```
sudo apt install curl
sudo apt install git
sudo apt-get install tree ctags vim
sudo apt install silversearcher-ag
sudo apt install libpcre3-dev
sudo apt install libssl-dev
sudo apt install libzip-dev
sudo apt install python3-pip
sudo apt install net-tools
```

# ubuntu下定时执行工具cron开启关闭重启

控制脚本一般为/etc/init.d/cron

```
启动：sudo /etc/init.d/cron start
关闭：sudo /etc/init.d/cron stop
重启：sudo /etc/init.d/cron restart
重新载入配置：sudo /etc/init.d/cron reload
```

可以用`ps aux | grep cron`命令查看`cron`是否已启动

# disable ssh login welcome message

```
sudo chmod -x /etc/update-motd.d/*
```

