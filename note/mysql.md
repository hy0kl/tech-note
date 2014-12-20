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

[src](http://www.oschina.net/question/123890_35887)比如一个评论表:id uid title content ctime取出最近的 10 条评论, 一条人发的多条评论的只取一条 ```sqlSELECT MAX(id) AS max_idFROM table1GROUP BY uidORDER BY max_id DESC 
```
