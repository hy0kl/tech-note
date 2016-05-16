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

