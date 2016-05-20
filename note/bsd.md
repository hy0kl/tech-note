# 初始化系统

## 加速 sshd 服务

```
# vi /etc/ssh/sshd_config

+ UseDNS no
```

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

