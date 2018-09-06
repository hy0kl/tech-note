# 实用SQL

## 按客户端IP分组,看哪个客户端的链接数最多

```
SELECT client_ip, COUNT(client_ip) AS client_num FROM
    (SELECT substring_index(host, ':', 1) AS client_ip FROM information_schema.processlist) AS connect_info
    GROUP BY client_ip ORDER BY client_num DESC;
```

## 查看正在执行的线程,并按Time倒排序

```
SELECT * FROM information_schema.processlist
WHERE Command != 'Sleep'
ORDER BY Time DESC;
```

## 找出所有执行时间超过5分钟的线程,拼凑出kill语句

```
SELECT CONCAT('kill ', id, ';') FROM information_schema.processlist
WHERE Command != 'Sleep' AND Time > 300
ORDER BY Time DESC;
```

## 统计库表的基本信息

```
SELECT TABLE_NAME, TABLE_ROWS, ENGINE, DATA_LENGTH, INDEX_LENGTH
FROM `information_schema`.`tables`  WHERE `table_schema` = 'DB_NAME';
```

## 统计库占用空间大小

```
SELECT SUM(DATA_LENGTH) + SUM(INDEX_LENGTH)
FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'DB_NAME';
```

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

#### 查询优化处理

- 语法解析器和预处理
- 查询优化器
    - SHOW STATUS LIKE 'Last_query_cost';
    - 导致MySQL优化器选择错误执行计划的原因:
        - 统计信息不准确
        - 执行计划中的成本估算不等同于实际执行的成本
        - MySQL的最优可能和你想的最优不一样
        - MySQL从不考虑其他并发执行的查询,这可能会影响到当前查询的速度
        - MySQL也并不是任何时候都是基于成本的优化
        - MySQL不会考虑不受其控制的操作的成本
        - 优化器有时候无法估算所有可能的执行计划,所有它可能错过实际上最优的执行计划
    - MySQL能够处理的优化类型
        - 重新定义关联表顺序
        - 将外连接转化为内连接
        - 使用等价变换规则
        - 优化`COUNT()`,`MIN()`和`MAX()`
        - 预估并转化常数表达式
        - 覆盖索引扫描
        - 子查询优化
        - 提前终止查询
        - 等值传播
        - 列表`IN()`的比较
    - 数据和索引的统计信息
    - 执行计划
    - 关联查询优化器
    - 排序优化
        - 两次传输排序(旧版本使用)
        - 单次传输排序(新版本使用)

            当查询需要所有列的总长度不超过参数`max_length_for_sort_data`时,MySQL使用"单次传输排序",可以通过这个参数影响MySQL排序算法的选择.


在关联操作中,范围检查的执行计划会针对每一行重新评估索引.可以通过`EXPLAIN`执行计划中的`Extra`列是否有`range checked for each record`来确认这一点.该执行计划还会增加`select_full_range_join`这个服务器变量的值.

#### 查询执行引擎

#### 返回结果给客户端

### MySQL查询优化器的局限性

#### 关联子查询

- 一般建议使用左外连接(LEFT OUTER JOIN)代替子查询,完成在一张表里面存在,在另一张表里面不存的查询

一是不需要听取那些关于子查询的"绝对真理",二是应该用测试来验证对子查询扫执行计划和响应时间的假设.

#### `UNION`的限制

#### 索引合并优化

#### 等值传递

#### 并行执行

#### 哈希关联

#### 松散索引扫描

#### 最大值和最小值优化

#### 在同一个表上查询和更新

### 查询优化器的提示(hint)

- `HIGH_PRIORITY`和`LOW_PRIORITY`

    只对使用表锁的存储引擎有效

- DELAYED
- `STRAIGHT_JOIN`
- `SQL_SMALL_RESULT`和`SQL_BIG_RESULT`
- `SQL_BUFFER_RESULT`
- `SQL_CACHE`和`SQL_NO_CACHE`
- `SQL_CALC_FOUND_ROWS`
- `FOR UPDATE`和`LOCK IN SHARE MODE`

    只对实现了行级锁的存储引擎有效.很容易造成服务器的锁争用问题.

- `USE INDEX`,`IGNORE INDEX`和`FORCE INDEX`

#### 控制优化器行为的参数

- `optimizer_search_depth`
- `optimizer_prune_level`
- `optimizer_switch`

### 优化特定类型的查询

#### 优化`COUNT()`查询
#### 优化关联查询
#### 优化子查询
#### 优化`GROUP BY`和`DISTINCT`
#### 优化`GROUP BY WITH ROLLUP`
#### 优化`LIMIT`
#### 优化`SQL_CALC_FOUND_ROWS`
#### 优化`UNION`查询

除非确实需要服务器消除重复的行,否则就一定要使用`UNION ALL`.如果没有`ALL`关键字,MySQL会给临时表加`DISTINCT`选项,会导致对整个临时表的数据做唯一性检查.

#### 静态查询分析
#### 使用用户自定义变量

- 优化排名语句
- 避免重复查询刚刚更新的数据
- 统计更新和插入的数量
- 确定取值的顺序
- 编写偷懒的`UNION`

基础原则:

- 尽量少做,可以的话就不要做任何事情.除非不得已,否则不要使用轮询,因为这会增加负载,而且还会带来很多低产出的工作.
- 尽可能快地完成需要做的事情.尽量使用`UPDATE`代替先`SELECT FOR UPDATE`再`UPDATE`的写法,因为整条提交的速度越快,持有锁时间就越短,可以大减少竞争和加速串行执行效率.将已经处理完成和未处理的数据分开,保证数据集足够小.
- 某些查询是无法优化的,考虑使用不同的查询或者不同的策略去实现相同的目的.

#### 使用自定义函数

#### 总结

优化通常需要三管齐下: 不做,少做,快速地做.

## MySQL高级特性

### 分区表

MySQL在创建表时使用`PARTITION BY`子句定义每个分区存放的数据.在执行查询的时候,优化器会根据分区定义过滤那些没有我们需要数据的分区,这样查询就无须扫描所有分区---只需要查找包含需要数据的分区.

在下面的场景中,分区可以起到非常大的作用:

- 表非常大以至于无法全部都放在内存中,或者只在表的最后部分有热点数据,其他均是历史数据.
- 分区表的数据更容易维护.可以对一个独立分区进行优化,检查,修复等操作.
- 分区表的数据可以分布在不同的物理设备上,从而高效的使用多个硬件设备.
- 可以使用分区表来避免某些特殊的瓶颈.
- 如果需要,还可以备份和恢复独立的分区,这在非常大的数据集的场景下效果非常好.

分区表的限制:

- 一个表最多只能有1024个分区.
- 在MySQL5.1中,分区表达式必须是整数,或者是返回整数的表达式.在MySQL5.5中,某些场景中可以直接使用列来进行分区.
- 如果分区字段中有主键或者唯一索引的列,那么所有主键列和唯一索引列都必须包含进来.
- 分区表中无法使用外键约束.

#### 分区表的原理

分区表上的操作按照下的操作逻辑进行:

- SELECT 查询

    当查询一个分区表的时候,分区层先打开并锁住所有的底层表,优化器先判断是否可以过滤部分区区,然后再调用对应的存储引擎接口访问各个分区的数据.

- INSERT 操作

    当写入一条记录时,分区层先打开并锁住所有的底层表,然后确定哪个分区接收这条记录,再将记录写入对应的底层表.

- DELETE 操作

    当删除一条记录时,分区层先打开并锁住所有的底层表,然后确定数据对应的分区,最后对相应底层表进行删除操作.

- UPDATE 操作

    当更新一条记录时,分区层先打开并锁住所有的底层表,MySQL先确定需要更新的记录在哪个分区,然后取出数据并更新,再判断更新后的数据应该放在哪个分区,最后对底层表进行写入操作,并对原数据所在的底层表进行删除操作.

#### 分区表的类型

```sql
CREATE TABLE sales (
    order_date DATATIME NOT NULL,
    -- other columns omitted
) ENGINE=InnoDB PARTITION BY RANGE(YEAR(order_date)) (
    PARTITION p_2010 VALUES LESS THAN (2010),
    PARTITION p_2011 VALUES LESS THAN (2011),
    PARTITION p_2012 VALUES LESS THAN (2012),
    PARTITION p_catchall VALUES LESS THAN MAXVALUE );
```

`PARTITION`分区子句中可以使用各种函数.但有一个要示,表达式返回的值要是一个确定的整数,且不能是一个常数.

MySQL还支持键值,哈希和列表分区,这其中有些还支持子分区,不过在生产环境中很少见到.在MySQL5.5中,还可以使用`RANGE COLUMNS`类型的分区,这样即使是基于时间的分区也无须再将其转化成一个整数.

其他分区技术:

- 根据键值进行分区,来减少InnoDB的互斥量竞争.
- 使用数学模函数来进行分区,然后将数据轮询放入不同的分区.
- 假设表有一个自增的主键列id,希望根据时间将最近的热点数据集中存.那么必须将时间戳包含在主键当中才行,而这和主键本身的意义相矛盾.这种情况下也可以使用这样的分区表达式来实现相同的目的: HASH(id DIV 1000000),这将为100万数据建立一个分区.这样一方面实现了当初的分区目的,另一方面比起使用时间范围分区还避免了一个问题,就是当超过一定阈值时,如果使用时间范围分区就必须新增分区.

#### 如何使用分区表

为了保证大数据量的可扩展性,一般有下面两个策略:

1. 全量扫描数据,不要任何索引.
1. 索引数据,分离热点.

#### 什么情况下会出问题

- NULL值会使分区过滤无效
- 分区列和索引列不匹配
- 选择分区的成本可能很高
- 打开并锁住所有底层表的成本可能很高
- 维护分区的成本很高

目前分区实现中的一些其他限制:

- 所有分区都必须使用相同的存储引擎
- 分区函数中可以使用的函数和表达式也有一些限制
- 某些存储引擎不支持分区
- 对于MyISAM的分区表,不能再使用`LOAD INDEX INFO CACHE`操作
- 对于MyISAM表,使用分区表时需要打开更多的文件描述符

### 查询优化

对于访问分区表来说,很重要的一点是要在`WHERE`条件中带入分区列,有时间即使看似多余的也要带上,这样就可以让优化器能够过滤掉无须访问的分区.如果没有这些条件,MySQL就需要让对应存储引擎访问这个表的所有分区,如果表非常大的话,就可能会非常慢.

使用`EXPLAIN PARTITION`可以观察优化器是否执行了分区过滤.

MySQL只能在使用分区函数的列本身进行比较时才能过滤分区,而不能根据表达式的值去过滤分区,即使这个表达式就是分区函数也不行.

一个很重要的原则是: 即便在创建分区时可以使用表达式,但在查询时却只能根据列来过滤分区.

### 合并表

一种被淘汰的技术,在未来的版本中可能被删除.

### 视图

MySQL使用两种算法来处理视图,分别称为合并算法(MERGE)和临时表算法(TEMPTABLE).

如果视图中包含`GROUP BY`,`DISTINCT`,任何聚合函数,`UNION`,子查询等,只要无法在原表记录和视图记录中建立一一映射的场景中,MySQL都将使用临时表算法来实现视图.

#### 可更新视图

所有使用临时表算法实现的视图都无法被更新.

MySQL不支持在视图上建任何触发器.

#### 视图对性能的影响

#### 视图的限制

`SHOW CREATE VIEW`

### 外键约束

InnoDB是目前MySQL中唯一支持外键的内置存储引擎.

使用外键是有成本的.

外键约束使用得查询需要额外访问一些别的表,这也意味着需要额外的锁.

### 在MySQL内部存储代码

MySQL允许通过触发器,存储过程,函数的形式来存储代码.

#### 优点

- 在服务器内部执行,离数据最近,另外服务器上执行还可以节省带宽和网络延迟.
- 是一种代码重用.可以方便地统一业务规则,保证某些行为总是一致,所以也可以为应用提供一定的安全性.
- 可以简化代码的维护和版本更新.
- 可以帮助提升安全,比如提供更细粒度的权限控制.
- 服务器端可以缓存存储过程的执行计划,这对于需要反复调用的过程,会大大降低消耗.
- 在服务器端部署,备份,维护都可以在服务器端完成.
- 可以在应用开发和数据库开发人员之间更好的分工.

#### 缺点

- MySQL本身没有提供好用的开发和调试工具,所以编写MySQL的存储代码比其他的数据库要更难些.
- 较之应用程序的代码,存储代码效率要稍微差些.
- 存储代码可能会给应用程序代码的部署带来额外的复杂性.
- 存储程序都部署在服务器内,所以可能有安全隐患.
- 存储过程会给数据库服务器增加额外的压力,而数据库服务器的扩展性相比应用服务器要差很多.
- MySQL并没有什么选项可以控制存储程序的资源消耗,所以在存储过程中一个小错误,可能直接把服务器拖死.
- 存储代码在MySQL中的实现有很多限制-执行计划缓存是连接级别的,游标的物化和临时表相同,在MySQL5.5版本之前,异常处理也非常困难,等等.简而言之,较之T-SQL或者PL/SQL,MySQL在存储代码功能还非常非常弱.
- 调试MySQL的存储过程是一件很困难的事情.
- 和基于语句的二进制日志复制合作得并不好.

#### 存储过程和函数

- 优化器无法使用关键字`DETERMINISTIC`来优化单个查询中多次调用存储函数的情况.

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

