# bsd pkg 安装 PostgreSQL

```
# pkg install postgresql95-client postgresql95-contrib postgresql95-server
# psql -U pgsql -d postgres
# psql --username=pgsql --dbname postgres
```

# 开启本地密码认证

```
# vim /usr/local/pgsql/data/pg_hba.conf

将`METHOD`由`trust`改为`md5`,重启服务:
# service postgresql restart
```

# 免密码登陆

## 方法一: 设置环境亦是

```
# export PGPASSWORD="登陆密码"
```

## 方法二: .pgpass

```
# vim ~/.pgpass
# chmod 0600 ~/.pgpass
# psql -U pgsql -d postgres -w
```

### .pgpass 密码文件格式

***hostname:port:database:username:password***

建议采用方法二,亲测有效

# [流式主从复制](http://www.cnblogs.com/yjf512/p/4499547.html)

## master 配置

`192.168.11.169`

```
## 1. 修改 pg_hba.conf，增加replica用户，进行同步
# vim pg_hba.conf

host    replication     replica     192.168.11.170/32                 md5

## 2. postgres设置密码
postgres# CREATE ROLE replica LOGIN REPLICATION ENCRYPTED PASSWORD 'replica';

## 3. 修改 postgresql.conf

wal_level = hot_standby  # 这个是设置主为wal的主机

max_wal_senders = 4 # 这个设置了可以最多有几个流复制连接，差不多有几个从，就设置几个
wal_keep_segments = 64 # 设置流复制保留的最多的xlog数目
wal_sender_timeout = 5s # 设置流复制主机发送数据的超时时间

max_connections = 100 # 这个设置要注意下，从库的max_connections必须要大于主库的

## 4. 重启服务
# service postgresql restart
```

## slave 配置

`192.168.11.170`

```
## 1. 同步源数据
### 注意同步文件权限属性,建议采用 sudo -u pgsql,或直接 su - pgsql 来同步命令操作
# pg_basebackup -P -D /usr/local/pgsql/data -h 192.168.11.169 -p 5432 -U replica -W
NOTICE:  WAL archiving is not enabled; you must ensure that all required WAL segments are copied through other means to complete the backup
报了NOTICE,没有影响启动服务

## 2. 创建和修改配置文件
# cd /usr/local/pgsql/data
# cp /usr/local/share/postgresql/recovery.conf.sample recovery.conf
# vi recovery.conf

recovery_target_timeline = 'latest' # 这个说明这个流复制同步到最新的数据
standby_mode = on  # 这个说明这台机器为从库
primary_conninfo = 'host=192.168.11.169 port=5432 user=replica password=replica'  # 这个说明这台机器对应主库的信息

## 3. 主配置文件 postgresql.conf

max_connections = 200 ＃ 一般查多于写的应用从库的最大连接数要比较大
hot_standby = on  # 说明这台机器不仅仅是用于数据归档，也用于数据查询
max_standby_streaming_delay = 30s # 数据流备份的最大延迟时间
wal_receiver_status_interval = 1s  # 多久向主报告一次从的状态，当然从每次数据复制都会向主报告状态，这里只是设置最长的间隔时间
hot_standby_feedback = on # 如果有错误的数据复制，是否向主进行反馈

## 4. 启动从服务器
# service postgresql start
```

## 查看主从复制状态

主库执行

```
postgres=# SELECT * FROM pg_stat_replication;
```

关注的点:

- `backend_start` 主从搭建的时间
- `state` 同步状态 startup: 连接中、catchup: 同步中、streaming: 同步
- `sync_priority` 同步Replication的优先度 0: 异步、1～?: 同步(数字越小优先度越高)
- `sync_state` 有三个值，async: 异步、sync: 同步、potential: 虽然现在是异步模式，但是有可能升级到同步模式

# 管理类SQL

## 数据库角色属性

```sql
/** 要创建一个带有登录权限的角色 */
CREATE ROLE user_name LOGIN;
/** 或 */
CREATE USER user_name;

/** 创建一个新数据库超级用户 */
CREATE ROLE super_user_name SUPERUSER;

/** 创建一个拥有创建数据库权限的角色 */
CREATE ROLE c_name CREATEDB LOGIN;

/** 创建一个可以给予权限到其他角色的角色 */
CREATE ROLE r_name CREATEROLE;

/** 创建一个可以发起流复制的角色 */
CREATE ROLE replica REPLICATION LOGIN;

/** 创建角色时提供口令 */
CREATE ROLE work LOGIN CREATEDB ENCRYPTED PASSWORD 'work';
CREATE ROLE admin LOGIN SUPERUSER  ENCRYPTED PASSWORD '10ve9ir1';
```

## 角色成员关系

[GRANT](http://www.postgres.cn/docs/9.5/sql-grant.html) [REVOKE](http://www.postgres.cn/docs/9.5/sql-revoke.html)

## 管理数据库

```sql
/** 确定现有数据库的集合 */
SELECT datname FROM pg_database;
SELECT datname FROM pg_database ORDER BY datname;

/** 创建一个数据库 */
CREATE DATABASE dev;
```

### [创建一个数据库](http://www.postgres.cn/docs/9.5/sql-createdatabase.html)

```
CREATE DATABASE name
    [ [ WITH ] [ OWNER [=] user_name ]
           [ TEMPLATE [=] template ]
           [ ENCODING [=] encoding ]
           [ LC_COLLATE [=] lc_collate ]
           [ LC_CTYPE [=] lc_ctype ]
           [ TABLESPACE [=] tablespace_name ]
           [ CONNECTION LIMIT [=] connlimit ] ]
```

# 一些常用SQL

```sql
/** 创建开发者账户 */
CREATE ROLE work LOGIN CREATEDB ENCRYPTED PASSWORD 'work';

CREATE DATABASE dev OWNER work;
/** user: work */
CREATE DATABASE test TEMPLATE template0;

SHOW client_encoding;
RESET client_encoding;
SET NAMES 'UTF8';

/** 查询所有用户的加密密码 */
SELECT usename, passwd FROM pg_shadow;
/** 查询指定用户的加密密码 */
SELECT usename, passwd FROM pg_shadow WHERE user IN('pgsql', 'work', 'zabbix');

/** 更改表字段类型 */
ALTER TABLE db_table ALTER COLUMN id TYPE bigint;
/** 自增id */
ALTER TABLE db_table ALTER COLUMN id SET DEFAULT nextval('db_table_id_seq'::regclass);

/** 删除索引 */
DROP INDEX IF EXISTS table_name_idx;
/** 删除主键 */
ALTER TABLE IF EXISTS table_name DROP CONSTRAINT IF EXISTS table_name_pkey;
/** 删除字段默认值 */
ALTER TABLE IF EXISTS table_name ALTER COLUMN id DROP DEFAULT;
/** 删除自增字段 */
DROP SEQUENCE IF EXISTS table_name_id_seq;
```

# 常用命令

```
\l[+]   [PATTERN]      list databases
\d[S+]                 list tables, views, and sequences
\d[S+]  NAME           describe table, view, sequence, or index
\df[antw][S+] [PATRN]  list [only agg/normal/trigger/window] functions
\dg[+]  [PATTERN]      list roles
\dp     [PATTERN]      list table, view, and sequence access privileges
\e [FILE] [LINE]       edit the query buffer (or file) with external editor
\p                     show the contents of the query buffer
\conninfo              display information about current connection
\x [on|off|auto]       toggle expanded output (currently off)
\encoding [ENCODING]   show or set client encoding
\timing [on|off]       toggle timing of commands (currently off)
\password [USERNAME]   securely change the password for a user
\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                       connect to new database
\o [FILE]              send all query results to file or |pipe
\h [NAME]              help on syntax of SQL commands, * for all commands
```

# 备忘录

- 自增序列`SERIAL`
- `timestamp`列指定默认值为`CURRENT_TIMESTAMP`，这样将得到行被插入时的时间
- 检查约束`CHECK`
- 命名的约束: 在约束名称标识符前使用关键词`CONSTRAINT`
- 非空约束: NOT NULL
- 唯一约束: UNIQUE
- 主键: PRIMARY KEY
- 外键: REFERENCES
  ON DELETE RESTRICT: 阻止删除一个被引用的行
  ON DELETE CASCADE: CASCADE指定当一个被引用行被删除后，引用它的行也应该被自动删除。
- [数据类型](http://www.postgres.cn/docs/9.5/datatype.html)
- [枚举类型](http://www.postgres.cn/docs/9.5/datatype-enum.html)可以使用[CREATE TYPE](http://www.postgres.cn/docs/9.5/sql-createtype.html)来创建
- 修改`pg_hba.conf`后`service postgresql reload`即可生效,但修改`postgresql.conf`需要`restart`
- 类型`decimal`和`numeric`是等效的。两种类型都是SQL标准的一部分。

# 高阶

## [Update timestamp when row is updated in PostgreSQL](http://stackoverflow.com/questions/1035980/update-timestamp-when-row-is-updated-in-postgresql)

### Q

In MySQL, we can execute this where it updates the column changetimestamp every time the row is changed:

```sql
create table ab (
  id int,
  changetimestamp timestamp NOT NULL default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP
);
```

Is there something similar to do the above in PostgreSQL?

### A

Create a function that updates the changetimestamp column of a table like so:

```sql
CREATE OR REPLACE FUNCTION update_changetimestamp_column()
RETURNS TRIGGER AS $$
BEGIN
   NEW.changetimestamp = now();
   RETURN NEW;
END;
$$ language 'plpgsql';
```

Create a trigger on the table that calls the `update_changetimestamp_column()` function whenever an update occurs like so:

```sql
CREATE TRIGGER update_ab_changetimestamp BEFORE UPDATE
    ON ab FOR EACH ROW EXECUTE PROCEDURE
    update_changetimestamp_column();
```

## [MySQL UNIX_TIMESTAMP for PostgreSQL](http://stackoverflow.com/questions/5857990/mysql-unix-timestamp-for-postgresql)

### Q

Basically I am trying to convert a date field from my database into a number of seconds old. I used UNIX_TIMESTAMP on our old MySQL database and am looking for an equivalent function in PostgreSQL.

### A

```sql
SELECT extract(epoch FROM your_datetime_column)
FROM your_table
```

More details in the manual:
http://www.postgresql.org/docs/current/static/functions-datetime.html#FUNCTIONS-DATETIME-EXTRACT

## [给PostgreSQL添加MySQL的unix_timestamp与from_unixtime函数](http://www.jiangmiao.org/blog/430.html)

MySQL的2个常用函数unix_timestamp()与from_unixtime PostgreSQL并不提供，但通过PostgreSQL强大的扩展性可以轻松的解决问题。
话说远在天边，尽在眼前，文档看仔细，问题迎仞解。PostgreSQL 题供extract与date_part取epoch即可:

```
unix_timestamp() = round(date_part('epoch',now()))
from_unixtime(int) = to_timestamp(int)
```

### 添加函数`unix_timestamp()`

```sql
CREATE FUNCTION unix_timestamp() RETURNS integer AS $$
SELECT (date_part('epoch',now()))::integer;
$$ LANGUAGE SQL IMMUTABLE;
```

### 添加函数`from_unixtime()`

```sql
CREATE FUNCTION from_unixtime(int) RETURNS timestamp AS $$
SELECT to_timestamp($1)::timestamp;
$$ LANGUAGE SQL IMMUTABLE;
```

# postgreSQL function for last inserted ID

[see](http://stackoverflow.com/questions/2944297/postgresql-function-for-last-inserted-id)

## Option 1: `CURRVAL(<sequence name>);`

```sql
INSERT INTO persons (lastname, firstname) VALUES ('Smith', 'John');

SELECT currval('persons_id_seq');
/** OR */
SELECT currval(pg_get_serial_sequence('persons', 'id'));
```

## Option 2:  `INSERT with RETURNING`

```sql
INSERT INTO persons (lastname,firstname) VALUES ('Smith', 'John') RETURNING id;
```

This is the most clean, efficient and safe way to get the id. It doesn't have any of the risks of the previous.

# 导入数据时报以下警告

```
WARNING:  no privileges could be revoked for "public"
WARNING:  no privileges could be revoked for "public"
WARNING:  no privileges were granted for "public"
WARNING:  no privileges were granted for "public"
```

# SQL错误后停止执行

```
$ psql --set ON_ERROR_STOP=on -U db_user -d db_name -f backup.sql
```

# [pg 序列Sequence](https://segmentfault.com/a/1190000002980194)

## 创建序列

```sql
CREATE SEQUENCE sequence_name
    [ INCREMENT increment ]     -- 自增数，默认是 1
    [ MINVALUE minvalue ]       -- 最小值
    [ MAXVALUE maxvalue ]       -- 最大值
    [ START start ]             -- 设置起始值
    [ CACHE cache ]             -- 是否预先缓存
    [ CYCLE ]                   -- 是否到达最大值的时候，重新返回到最小值
```

## 查看序列

```sql
\ds

/** 查看详细 Sequence 信息 */
\d table_name_id_seq

/** 查询下 seq 的 last_value（当前的序列值）和 increment_by (当 nextval() 被调用，当前值将会被增加) */
SELECT last_value, increment_by FROM table_name_id_seq;
```

## Sequence 的函数

```
nextval('sequence_name'): 将当前值设置成递增后的值，并返回
currval('sequence_name'): 返回当前值
setval('sequence_name', n, b=true): 设置当前值；b 默认设置 true，下一次调用 nextval() 时，直接返回 n，如果设置 false，则返回 n+increment:
```

## 清空表,并重置序列发生器

```sql
TRUNCATE table_name RESTART IDENTITY;
```

# 最佳实践

- 建议使用ip4,ip4r,ip6,ip6r,ipaddress,iprange 来存储IP,IP范围；使用macaddr来存储MAC (Media Access Control) address
- 建议使用`timestamp with time zone(timestamptz)`,而不用`timestamp without time zone`,避免时间函数在对于不同时区的时间点返回值不同,也为业务国际化扫清障碍
- 建议使用`NUMERIC(precision, scale)`来存储货币金额和其它要求精确计算的数值, 而不建议使用`real`,`double precision`
- 建议使用`hstore`来存储非结构化,`key-value`键值型,对数不定的数据
- 建议使用`jsonb`(比json更有优势)来存储`JSON (JavaScript Object Notation) data`
- 建议主键的一步到位的写法: `id serial primary key`或`id bigserial primary key`
- 建议内容系统中size较大的table主键的等效写法如下,便于后续维护：

  ```
  create table tbl_name(id serial not null);
  create unique index CONCURRENTLY ON tbl_name (id);
  ```

- PostgreSQL 提供的`index`类型: `B-tree`,`Hash`,`GiST (Generalized Search Tree)`,`SP-GiST (space-partitioned GiST)`,`GIN (Generalized Inverted Index)`,`BRIN (Block Range Index)`,目前不建议使用`Hash`
- 建议`create`或`drop index`时,加`CONCURRENTLY`参数,这是个好习惯，达到与写入数据并发的效果
- 建议对于频繁`update`,`delete`的包含于`index`定义中的`column`的`table`, 用`create index CONCURRENTLY`,`drop index CONCURRENTLY`的方式进行维护其对应`index`
- 建议用`unique index`代替`unique constraints`,便于后续维护
- 建议对`where`中带多个字段`and`条件的高频`query`，参考数据分布情况，建多个字段的联合`index`
- 建议对固定条件的（一般有特定业务含义）且选择比好（数据占比低）的`query`，建带`where`的`Partial Indexes`

  ```
  select * from test where status=1 and col=?; -- 其中status=1为固定的条件
  create index on test (col) where status=1;
  ```

- 建议对经常使用表达式作为查询条件的query，可以使用表达式或函数索引加速query

  ```
  select * from test where exp(xxx);
  create index on test ( exp(xxx) );
  ```

- 建议不要建过多`index`，一般不要超过6个，核心table（产品，订单）可适当增加`index`个数
- 建议清空表时，使用`truncate`，不建议使用`delete`
- 建议执行`DDL`,比如`CRAETE`、`DROP`、`ALTER`等, 尤其多条， 不要显式的开`transaction`, 因为加`lock`的`mode`非常高,极易产生`deadlock`

# PostgreSQL数据库内核分析

## part 2

PostgreSQL的名字空间层次是: 数据库.模式.表.属性.

当访问一个对象时,PostgreSQL会按以下空间顺序进行搜索:

- 特殊名字空间(special),仅用于创建模式
- 临时表的名字空间(TEMP)
- 系统表的名字空间

在PostgreSQL源代码`src/include/catalog`子目录下有一个sell脚本`unused_oids`用来输出当前版本中预分配和预留的OID的使用情况.

初始化数据集簇包括创建包含数据库系统所有数据的数据目录、创建共享的系统表、创建其他的配置文件和控制文件，并创建三个数据库：模板数据库 template1 和 template0 、默认的用户数据库postgres。以后用户创建一个新数据库时，template1 数据库里的所有内容（包括系统表文件）都会拷贝过来，因此，任何在 template1 里面安装的内容都自动拷贝到之后创建的数据库中。template0 和 postgres 都是通过拷贝 template1 创建的。

对于某个具体的数据库,在`PGDATA/base`里都对应一个子目录,子目录的名字是该数据库在系统表`pg_database`里的OID.

在PostgreSQL中，默认情况下会将数据文件存在 PGDATA 指定的目录下，但如果 PGDATA 所在磁盘空间不足或者用户出于磁盘性能考虑，需要为系统增加新的物理存储位置（比如在另外一个磁盘上的目录），则可以使用“表空间”来进行扩展。表空间从物理意义上来说就是一个新的磁盘目录（当然这个目录可以和 PGDATA 同处一个磁盘或者不同磁盘），指定放在表空间中的数据库对象的物理文件都存放在表空间对应的目录中。如果某个数据库中的数据库对象被指定放在一个表空间中，那么在表空间的目录中会以该数据库的 OID 为名称创建一个目录，该数据库中属于该表空间的对象的物理文件都放在这个目录中。每个用户定义的表空间在`PGDATA/pg_tbspc`目录里面都有一个符号链接，它指向表空间的物理目录，该符号链接用表空间的 OID 命名。系统默认创建的表空间`pg_default`和`pg_global`并没有通过符号链接的方式指向其物理目录，而是直接对应`PGDATA/base`和`PGDATA/global`。

GUC(Grand Unified Configuration)模块实现了多种数据类型（有boolean、int、float、string）的变量配置。

字符串`ListenAddresses`：该字符串中存储的是服务器 IP 地址，如果是多接口主机的话，IP 地址之间用“,”隔开，“\*”表示所有可用 IP 接口，缺省值为“localhost”。

## part 3

### 外存管理

外存管理由SMGR(主要代码在smgr.c中)提供对外存操作的统一接口.

VFD: 虚拟文件描述符<br/>
FSM: 空闲空间映射表文件<br/>
VM: 可见性映射表文件

### 磁盘管理器

磁盘管理器是SMGR的一种具体实现,它对外提供了管理磁盘介质的接口,其主要实现在文件`md.c`中.磁盘管理器并非对磁盘上的文件直接进行操作,而是通过VFD机制来进行文件操作.

### 大数据存储

- TOAST(The Oversized-Attribute Storage Technique,超尺寸字段存储技巧)

  TOAST机制的实现代码在`src/backend/access/heap/tuptoaster.c`文件中.

  只有在准备向支持TOAST的属性中存储超过`BLCKSZ / 4`(通常是2KB)的数据时,TOAST机制才会被触发.

- 大对象

  - 二进制大对象(BLOB)
  - 字符大对象(CLOB)
  - 双字节字符大对象(DBCLOB)

### 内存管理

MemoryContext内存上下文

# 基本pg的分布式方案

[citus](https://github.com/citusdata/citus)

[GreenPlum](https://github.com/greenplum-db/gpdb)

[PGPool-II](https://github.com/pgpool/pgpool2)

[Postgres-XL](https://github.com/Postgres-XL/Postgres-XL) [postgres-x2](https://github.com/postgres-x2/postgres-x2)

[PL/Proxy](https://plproxy.github.io/)
