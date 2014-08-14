# 记一次数据不完整的事故

现象描述: 一个 JSON 数据接口,用浏览器访问,数据正常.但通过 php curl 的脚本来调用时,总是返回不完整的数据.确认不存在代码改动的干扰,查无果.后来发现开发机磁盘满了,却没有证据表明是直接原因.查看 nginx error log 时,发现如下记录:

```
2014/05/30 14:31:30 [crit] 11046#0: *81985 writev() "/usr/local/webserver/nginx/fastcgi_temp/9/66/0000010669" failed (28: No space left on device) while reading upstream, client: 192.168.25.32, server: , request: "POST /api/countries HTTP/1.1", upstream: "fastcgi://unix:/tmp/php-cgi.sock:", host: "192.168.3.200:8086"
```

由于确认是磁盘满导致的.
但遗憾的是,出问题的开发机把 php notice 关了,结果有用的错误信息被淹没在大量 php notice 错误之中了.可见开发环境关掉 notice 是多么屌爆的一件事.
