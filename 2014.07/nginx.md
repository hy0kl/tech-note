# nginx 日志切割

[参考](http://www.nginx.cn/255.html)

nginx的日志文件没有rotate功能。如果你不处理，日志文件将变得越来越大，还好我们可以写一个nginx日志切割脚本来自动切割日志文件。

第一步就是重命名日志文件，不用担心重命名后nginx找不到日志文件而丢失日志。在你未重新打开原名字的日志文件前，nginx还是会向你重命名的文件写日志，linux是靠文件描述符而不是文件名定位文件。

第二步向nginx主进程发送USR1信号。

nginx主进程接到信号后会从配置文件中读取日志文件名称，重新打开日志文件(以配置文件中的日志名称命名)，并以工作进程的用户作为日志文件的所有者。

重新打开日志文件后，nginx主进程会关闭重名的日志文件并通知工作进程使用新打开的日志文件。

工作进程立刻打开新的日志文件并关闭重名名的日志文件。

然后你就可以处理旧的日志文件了。

nginx日志按日期自动切割脚本如下

```shell
# nginx日志切割脚本
# author: http://www.nginx.cn

#!/bin/bash
# 设置日志文件存放目录
logs_path="/usr/local/nginx/logs/"
# 设置pid文件
pid_path="/usr/local/nginx/nginx.pid"

# 重命名日志文件
mv ${logs_path}access.log ${logs_path}access_$(date -d "yesterday" +"%Y%m%d").log

# 向nginx主进程发信号重新打开日志
kill -USR1 `cat ${pid_path}`
```

保存以上脚本nginx_log.sh,或者点此下载

crontab 设置作业

0 0 * * * bash /usr/local/nginx/nginx_log.sh

这样就每天的0点0分把nginx日志重命名为日期格式，并重新生成今天的新日志文件。

# 隐藏Nginx版本号

[参考](http://blog.cunss.com/?p=299)

一、 关于Nginx版本隐藏，有2种方法：

 1.编译安装时，可以源码包里面修改

 具体在 src/core/nginx.h 文件下

```c
#define nginx_version      1004004
#define NGINX_VERSION      "1.0.5"
#define NGINX_VER          "nginx/" NGINX_VERSION
```

修改这里面的版本号，当然可以伪造为其他，然后再进行编译

2.不想重新编译，可直接在Nginx配置文件中进行修改

在http{}段添加全局参数：

```conf
server_tokens off;
```

如果使用Tengine，除了完全兼容Nginx外，还有其独有的配置方法

同样在http{}段

使用全局参数：

```conf
server_tag off;
```

当然可以对server_tag 进行伪造，如

server_tag BAT/1.1;
以下为设置前后对比：

off状态

```
[root@cunss ~]# curl -I http://blog.cunss.com/
HTTP/1.1 200 OK
Date: Fri, 11 Apr 2014 15:53:50 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Pingback: http://blog.cunss.com/xmlrpc.php
```

设置为其他值

```
[root@cunss ~]# curl -I http://blog.cunss.com/
HTTP/1.1 200 OK
Server: BAT/1.1
Date: Fri, 11 Apr 2014 15:54:04 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Pingback: http://blog.cunss.com/xmlrpc.php
```

二、除了Nginx，apache也可以隐藏版本信息

需要在httpd.conf加入

```conf
ServerTokens ProductOnly
ServerSignature Off
```

三、同时wordpress使用php,因此php也需要进行相关安全设置

在php.ini中

```ini
expose_php = Off  #隐藏php版本号
disable_functions = system,exec,shell_exec,passthru,popen,dl,phpinfo #禁用危险函数
display_errors = Off    #关闭错误日志
allow_url_fopen = Off   # 禁用打开远程url
safe_mode = On     #打开php的安全模式
```


