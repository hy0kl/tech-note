# 高频命令 cli

# 生成 ssh key

```shell
$ ssh-keygen -t RSA -b 4096 -C "your_email@example.com"
```

# 进程和服务端口相关

```
lsof -p pid 可以看到当前进程所调用的文件
netstat -nlp # linux 显示监听,进程
netstat -anp # linux 查看哪些端口被打开
netstat -anp | grep tcp
sudo nmap -sT -O localhost # 更靠谱的查看那些端口被打开
lsof -ni | grep LISTEN # mac 下类似上条命令效果
lsof -i:9000 # 查看9000端口的占用者
```

# mysql 相关

```
unix_timestamp(now())
FROM_UNIXTIME( 1249488000, '%Y-%m-%d %H:%i' )
mysqli_real_escape_string()
```

# php

```
file_put_contents FILE_APPEND
htmlspecialchars ENT_QUOTES
```

# 日志级别

```
debug、info、notice、warning、error、critical 和 alert
```

# DNS

```
whois dig nslookup
```

# grep 去掉注释和空行

```
cat config | grep -v '^#' --color=no | grep -v '^$'
```

# MySQL
## mysql 仅导出库表结构为xml

```
mysqldump --xml --opt -d -u db_user  db_name -p > db_name.xml
```

## 只导出数据

```
mysqldump -t -u db_user db_name -p > db_name.sql
```

# PostgreSQL

## 导出数据

```
# 仅导出原始表结构
pg_dump -U dbuser -d dbname -h HOST -p PORT -c -s -f dbname.sql
# 导出结构和数据,以 INSERT 替代 COPY
pg_dump -U dbuser -d dbname -h HOST -p PORT --inserts -f dbname.sql
pg_dump -U dbuser -d dbname -h HOST -p PORT --inserts --if-exists -f dbname.sql
pg_dump -U dbuser -d dbname -h HOST -p PORT -c -C --column-inserts --if-exists -f dbname.sql
```

## `pg_dump`主要参数说明:

参数 | 说明
---- | ---
-c, --clean | clean (drop) database objects before recreating
-C, --create | include commands to create database in dump
-s, --schema-only | dump only the schema, no data
-t, --table=TABLE | dump the named table(s) only
--column-inserts | dump data as INSERT commands with column names
--if-exists | use IF EXISTS when dropping objects
--inserts | dump data as INSERT commands, rather than COPY

## 时间戳相关

```
unix_timestamp() = round(date_part('epoch',now()))
from_unixtime(int) = to_timestamp(int)
```

# gcc

```
gcc -ldl -lmath
```

# How do I find my RSA key fingerprint?

## Linux/BSD

```
$ ssh-keygen -lf /path/to/ssh/key
```

## MacOS

```
$ ssh-keygen -E md5 -lf /path/to/ssh/key
```

