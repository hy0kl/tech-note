# MySQL事务隔离级别

隔离级别 | 脏读(Drity Read) | 不可重复读(Non-repeatable read) | 幻读(Phantom Read)
--- | --- | --- | ---
Read Uncommitted（读取未提交内容） | v | v | v
Read Committed（读取提交内容） | x | v | v
Repeatable Read（可重读） | x | x | v
Serializable（可串行化） | x | x | x

# 高性能MySQL拾遗

## Schema与数据类型优化

- 如果`EXPLAIN`执行计划的`Extra`列包含"Using temporary",则说明这个查询使用了隐式临时表
- 如果使用的是InnoDB存储引擎,将不能在数据类型不是完全匹配的情况下创建外键,否则会有报错信息:`ERROR 1005(HY000): Can't create table`
- MySQL提供`INEF_ATON()`和`INEF_NTOA()`函数,实现IPv4地址和32位无符号整数的互相转换

## 创建高性能的索引

### 高性能的索引策略

1. 独立的列

  "独立的列"是指索引列不能是表达式的一部分,也不能是函数的参数

1. 前缀索引和索引选择性

  方法一: `SELECT COUNT(*) AS cnt, LEFT(cl, 7) AS pref FROM tbn GROUP BY pref ORDER BY cnt DESC LIMIT 10`

  方法二: `SELECT COUNT(DISTINCT cl) / COUNT(*) FROM tbn`

1. 多列索引
1. 选择合适的索引列顺序

  经验法则: 将选择性最高的列放到索引最前列

1. 聚族索引

### 覆盖索引

被索引覆盖的查询,也叫做索引覆盖查询

MySQL不能在索引中执行`LIKE`操作.这是底层存储引擎API的限制,MySQL5.5和更早的版本中只请允许在索引中做简单的比较操作(例如等于,不等于以及大于).MySQL能在索引中做最左前缀的匹配`LIKE`比较,因为该操作可以转换为简单的比较操作,但是如果是通配符开头的`LIKE`查询,存储引擎就无法比较匹配.

延时关联(deferred join)

如果`EXPLAIN`出来的`type`列的值主"index",则说MySQL使用了索引扫描来做排序

MySQL可以使用同一个索引既满足排序,又用于查找行.因此,如果可能,设计索引时应该尽可能地同时满足这两种任务,这样是最好的.

冗余索引和重复索引

`EXPLAIN`的`Extra`列出现了`Using where`,表示MySQL服务器将存储引擎返回行以后再应用`WHERE`过滤条件

InnoDB在二级索引上使用共享(读)锁,但访问主键索引需要排他(写)锁.


### 索引案例学习

1. 支持多种过滤条件

    在有更多不同值的列上创建索引的选择性会更好

1. 考虑其他常见的`WHERE`条件的组合

    我们总是尽可能让MySQL使用更多的索引列,因为查询只能使用索引的最左前缀,直到遇到第一个范围条件列.

    可以使用`IN()`来代替范围查询

1. 避免多个范围条件

    对于范围条件查询,MySQL无法再使用范围列后面的其他索引了,但对于"多个等值条件查询"则没有这个限制.

1. 优化排序

    例子: 高效使用(sex, rating)索引进行排序和分页:

    ```sql
    MySQL> SELECT <cols> FROM profiles INNER JOIN (
        ->  SELECT <primary key cols> FROM profiles
        ->  WHERE x.sex = 'M' ORDER BY rating 100000, 10
        -> ) AS x USING(<primary key cols>);
    ```

1. 维护索引和表
    1. 找到并修复损坏的表
    1. 更新索引统计信息
    1. `ANALYZE TABLE`
    1. 使用`SHOW INDEX FROM`命令来查看索引的基数(Cardinality)
    1. InnoDB会在表首次打开,或者执行`ANALYZE TABLE`,抑或表的大小发生非常大变化(大小变化超过十六分之一或者新插入了20亿行都会触发)的时候计算索引的统计信息.
    1. InnoDB在打开某些`INFORMATION_SCHMA`表,或者使用`SHOW TABLE STATUS`和`SHOW INDEX`,抑或在MySQL客户端开启自动补全功能的时候都会触发索引统计信息的更新.客户端或监控程序触发索引信息采样更新时会导致大量的锁,给服务器带来很多的额外压力.可以关闭`innodb_stats_on_metadata`参数来避免.
    1. 减少索引和数据的碎片.
        - 行碎片(Row fragmentation)
        - 行间碎片(Intra-row fragmentation)
        - 剩余空间碎片(Free space fragmentation)

## 查询性能优化

查询优化,索引优化,库表结构优化需要齐头并进,一个不落.

### 慢查询的基础: 优化数据访问

1. 是否向数据库请求了不需要的数据
    - 查询不需要的记录
    - 多表关联时返回全部列
    - 总是取出全部列
    - 重复查询相同的数据

### MySQL是否在扫描额外的记录

- 响应时间

    响应时间既可能是一个问题的结果也可能是一个问题的原因

- 扫描的行数
- 返回的行数

    扫描的行数和访问类型<br />
    在`EXPLAIN`语句中的`type`列反应了访问类型.访问类型有很多种,从全表扫描到索引扫描,范围扫描,唯一索引查询,常数引用等,速度从慢到快,扫描的行数也是从小到大.

一般MySQL能使用如下三种方式应用`WHERE`条件,从好到坏依次为:

- 在索引中使用`WHERE`条件来过滤不匹配的记录.这是在存储引擎层完成的.
- 使用索引覆盖扫描(在`Extra`列中出现了`Using index`)来返回记录,直接从索引中过滤不需要的记录并返回命中的结果.这是在MySQL服务器层完成的,但无须再回表查询记录.
- 从数据表中返回数据,然后过滤不满足条件的记录(在`Extra`列中出现`Using where`).这在MySQL服务器层完成,MySQL需要从数据表读出记录后过滤.

### 重构查询方式

1. 一个复杂查询还是多个简单查询
1. 切分查询
1. 分解关联查询
    - 让缓存效率更高
    - 将查询分解后,执行单个查询可以减少锁的竞争
    - 在应用层做关联,可以更容易对数据库进行拆分,更容易做到高性能和扩展
    - 查询本身效率也可能会提升
    - 可以减少冗余记录的查询

### 查询执行的基础

- 查询状态 `SHOW FULL PROCESSLIST`
    - Sleep 线程正在等待客户端发送新的请求
    - Query 线程正在执行查询或者正在将结果发送给客户端
    - Locked 在MySQL服务层,该线程正在等待表锁
    - Analyzing and statistics 线程正在收集存储引擎的统计信息,并生成查询的执行计划
    - Copying to tmp table [on disk] 线程正在执行查询,要么是文件排序操作,或者是`UNION`操作.如果这个状态后面还有"on disk"标记,那表示MySQL正在将一个内存临时表放到磁盘上
    - Sorting result 线程正在对结果集进行排序
    - Sending data  这表示多种情况: 线程可能在多个状态之间传送数据,或者在生成结果集,或者在向客户端返回数据

#### 查询缓存



# INSERT 时防止出现主键冲突错误的方法

[参考](http://hi.baidu.com/ytjwt/item/9169ce144caa92f8746a840d)

```
最近几天，产品上线比较多，从内网测试库导出表的部分内容到线上也就比平时频繁多了，这时候可能会出现主键冲突：
Error Code : 1062
Duplicate entry '1' for key 'PRIMARY'
总结下，三种解决方案来避免出错
1.insert ignore into
遇主键冲突，保持原纪录
mysql> select * from device ;
+-------+--------+-------------+
| devid | status | spec_char   |
+-------+--------+-------------+
|     1 | dead   | zhonghuaren |
|     2 | dead   | zhong       |
+-------+--------+-------------+
2 rows in set (0.00 sec)

mysql> insert into device values (1,'alive','yangting');
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'

mysql> insert ignore  into device values (1,'alive','yangting');
Query OK, 0 rows affected (0.00 sec)

mysql> select * from device ;
+-------+--------+-------------+
| devid | status | spec_char   |
+-------+--------+-------------+
|     1 | dead   | zhonghuaren |
|     2 | dead   | zhong       |
+-------+--------+-------------+
2 rows in set (0.00 sec)
可见 insert ignore  into当遇到主键冲突时，不跟改原纪录，也不报错

2：REPLACE into
遇主键冲突，替换原纪录，即 先删除原纪录，后insert 新纪录

mysql> replace  into device values (1,'alive','yangting');
Query OK, 2 rows affected (0.00 sec)

mysql> select * from device ;
+-------+--------+-----------+
| devid | status | spec_char |
+-------+--------+-----------+
|     1 | alive  | yangting  |
|     2 | dead   | zhong     |
+-------+--------+-----------+
2 rows in set (0.00 sec)


3：INSERT ... ON DUPLICATE KEY UPDATE
其实这个是原本需要执行3条SQL语句（SELECT,INSERT,UPDATE），缩减为1条语句即可完成。
即
IF (SELECT * FROM where 存在) {
UPDATE  SET  WHERE ;
} else {
INSERT INTO;
}
如：mysql> insert into device values (1,'readonly','yang') ON DUPLICATE KEY UPDATE status ='drain';
Query OK, 2 rows affected (0.00 sec)

上面语句伪代码表示即为
if (select * from device where devid=1)
{ update device set status ='drain' where devid=1
}else {insert into device values (1,'readonly','yang')}
很明显，devid=1  是有的，这样就执行update操作
mysql> select * from device ;
+-------+--------+-----------+
| devid | status | spec_char |
+-------+--------+-----------+
|     1 | drain  | yangting  |
|     2 | dead   | zhong     |
+-------+--------+-----------+
2 rows in set (0.00 sec)


测试表：
CREATE TABLE `device` (
`devid` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
`status` enum('alive','dead','down','readonly','drain') DEFAULT NULL,
`spec_char` varchar(11) DEFAULT '0',
PRIMARY KEY (`devid`)
) ENGINE=InnoDB
```

#  mysql 字段加索引, where 子句中用 > 比较时无法使用索引

```
使用 >=, <, = 时可以正确的使用索引.
指定以主键排序时可以使用到索引.
有了主键就需要合理使用.
```

# mysql 取表字段

```
方法一:
SELECT COLUMN_NAME
FROM information_schema.columns
WHERE table_name = '要查询的表';

方法二:
SELECT  COLUMN_NAME as '列名', DATA_TYPE as '字段类型', COLUMN_TYPE as '长度加类型'
FROM information_schema.`COLUMNS`
WHERE TABLE_SCHEMA LIKE '库名'
AND TABLE_NAME LIKE '表名';

方法三:
DESC `表名`;
DESCRIBE `表名`;
```

# mysql 先排序再分组

[src](http://www.oschina.net/question/123890_35887)

比如一个评论表:
id uid title content ctime

取出最近的 10 条评论, 一条人发的多条评论的只取一条

```sql
SELECT MAX(id) AS max_id
FROM table1
GROUP BY uid
ORDER BY max_id DESC
```

# mysql 报错`1030 Got error 28 from storage engine`

[see](https://stackoverflow.com/questions/10631387)

```
MySQL error "28 from storage engine" - means "not enough disk space".

MySQL-server# df -h
```

# MySQL SHOW 语法

## SHOW CHARACTER SET Syntax

```
SHOW CHARACTER SET
    [LIKE 'pattern' | WHERE expr]

SHOW CHARACTER SET LIKE 'utf%'
```

## SHOW COLUMNS Syntax

```
SHOW [FULL] {COLUMNS | FIELDS}
    {FROM | IN} tbl_name
    [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]

SHOW COLUMNS FROM statistics_diary
```

## `SHOW CREATE TABLE tbl_name`

## SHOW DATABASES Syntax

```
SHOW {DATABASES | SCHEMAS}
    [LIKE 'pattern' | WHERE expr]
```

## SHOW ERRORS Syntax

```
SHOW ERRORS [LIMIT [offset,] row_count]
SHOW COUNT(*) ERRORS
```

## SHOW GRANTS Syntax

```
SHOW GRANTS [FOR user]

SHOW GRANTS FOR 'jeffrey'@'localhost';
```

## SHOW INDEX Syntax

```
SHOW {INDEX | INDEXES | KEYS}
    {FROM | IN} tbl_name
    [{FROM | IN} db_name]
    [WHERE expr]
```

## `SHOW MASTER STATUS`

## SHOW OPEN TABLES Syntax

```
SHOW OPEN TABLES
    [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]
```

## `SHOW PRIVILEGES`

## `SHOW [FULL] PROCESSLIST`

## SHOW SLAVE STATUS Syntax

```
SHOW SLAVE STATUS [FOR CHANNEL channel]
```

## SHOW STATUS Syntax

```
SHOW [GLOBAL | SESSION] STATUS
    [LIKE 'pattern' | WHERE expr]

SHOW STATUS LIKE 'Key%';
```

## SHOW TABLE STATUS Syntax

```
SHOW TABLE STATUS
    [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]
```

## SHOW TABLES Syntax

```
SHOW [FULL] TABLES
    [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]
```

## SHOW VARIABLES Syntax

```
SHOW [GLOBAL | SESSION] VARIABLES
    [LIKE 'pattern' | WHERE expr]
```

## SHOW WARNINGS Syntax

```
SHOW WARNINGS [LIMIT [offset,] row_count]
SHOW COUNT(*) WARNINGS
```

