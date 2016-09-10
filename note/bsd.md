# 初始化系统

## 禁掉 sendmail 服务,有安全隐患

```
# vi /etc/rc.conf
# # 加入以下内容
sendmail_enable="NONE"
sendmail_submit_enable="NO"
sendmail_outbound_enable="NO"
sendmail_msp_queue_enable="NO"

重启机器,或
# killall sendmail
```

## 安装依赖

```
# cd /usr/ports/ports-mgmt/pkg
# make install clean
# pkg install --yes tree ctags gettext git wget curl gcc bash gmake pcre libxml2 libxslt
```

## 标准开发环境配置

```
$ cd ~
$ git clone https://github.com/hy0kl/profile.git
$ cd profile/
$ ./install.sh

$ login # OR 退出重登陆
```

# [配置静态 ip](https://wiki.freebsdchina.org/faq/networking)

## 即时生效

```
# ifconfig em0 192.168.0.200 255.255.254.0
# ifconfig em0 inet 192.168.0.201 netmask 255.255.254.0 alias   # 增加第二个ip
# route add default 192.168.0.1
```

## 永久生效

```
# vim /etc/rc.conf

ifconfig_em0="inet 192.168.0.200  netmask 255.255.254.0"
ifconfig_em0_alias0="inet 192.168.0.201  netmask 255.255.254.0"
defaultrouter="192.168.0.1"
```

# DNS 配置

```
# vim  /etc/resolv.conf

nameserver 114.114.114.114
nameserver 8.8.8.8
```

# [时区与时间同步](https://www.freebsdchina.org/forum/topic_50795.html)

```
# tzsetup # 安装完系统后设定时区
## 校对时间,建议系统初始时执行,以免影响依赖时间的服务
# ntpdate -t 3 -b asia.pool.ntp.org
```

## 开机自动校对时间

```shell
# vim /etc/rc.local
/usr/sbin/ntpdate -t 3 -b asia.pool.ntp.org

# chmod +x /etc/rc.local
```

# pkg 无法更新

```
## 查看更新记录
# pkg updating | less
## 强制更新
# pkg update -f
```

# BSD下运维小工具,防止重复劳动

## `relay`机`sshd`建议配置

编辑`/etc/ssh/sshd_config`

```
# 加速
UseDNS no
# ssh 禁止 root 远程登陆
PermitRootLogin no
# RSA认证
RSAAuthentication yes
# 开启公钥验证
PubkeyAuthentication yes
# 验证文件路径
AuthorizedKeysFile .ssh/authorized_keys
# 禁止密码认证
PasswordAuthentication no
# 禁止空密码
PermitEmptyPasswords no
# 禁用PAM
UsePAM no

# 配置完毕重启sshd
# service sshd restart
```

## `sshd`绑定`ip`

`ListenAddress`参数确定sshd监听的ip地址

# BSD 手工编译安装 Redis

- 记得用 gmake
- 修改`PREFIX`


# 加速 pkg 安装

改为台湾的源,能有效的加速

```
# vim /etc/pkg/FreeBSD.conf

亚洲的源: pkg0.twn.freebsd.org
```

# sqlmap error

missing one or more core extensions ('gzip', 'ssl', 'sqlite3', 'zlib') most probably because current version of Python has been built without appropriate dev packages (e.g. 'libsqlite3-dev')

```
## FreeBSD 10.1-RELEASE
# pkg install py27-sqlite3
```

# pkg 错误

pkg: uk-libgd-2.2.3,1 conflicts with libgd-2.1.1,1 (installs files into the same place).  Problematic file: /usr/local/bin/annotate

解决方法:

```
## 升级
# pkg upgrade libgd
```
