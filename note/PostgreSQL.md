# PostgreSQL

## 主键

UNIQUE NOT NULL 等价于 PRIMARY KEY

FOREIGN KEY 外键

REFERENCES 约束外键

ON DELETE RESTRICT  -- 禁止删除被引用的行  
ON DELETE CASCADE   -- 在删除一个被引用的行的时候，所有引用它的行也会被自动删除掉

与ON DELETE类似的还有ON UPDATE选项， 它是在被引用字-段修改(更新)的时候调用的，可用的动作是一样的。

# 其他

CONSTRAINT 命名约束

## Tips

- 添加一个字段并填充缺省值将会导致更新表中的所有行(为了存储新字段的值)， 但如果没有声明缺省值，PostgreSQL就可以 避免物理更新。所以如果你将要在新字段中填充的值大多数都不等于缺省值， 那么最好添加一个没有缺省值的字段，然后再使用UPDATE更新数据。

- 到底多大的表会从分区中收益取决于具体的应用， 不过有个基本的拇指规则就是表的大小超过了数据库服务器的物理内存大小。
- `DROP TABLE IF EXISTS`用来避免错误消息，不过这并不符合 SQL 标准。

## 权限

```sql
GRANT UPDATE ON table_name TO db_user;
/** 在权限的位置写上 ALL 则赋予所有与该对象类型相关的权限 */

REVOKE ALL ON table_name FROM db_user;
/** 对象所有者的特殊权限(也就是，DROP，GRANT， REVOKE，等权限)总是隐含地属于所有者，并且不能赋予或者撤销。 但是对象所有者可以选择撤销自己的普通权限，比如把一个表做成对自己和别人都是只读的。 */
```

## 连接表

```
交叉连接(笛卡尔积)
    T1 CROSS JOIN T2
    FROM T1 CROSS JOIN T2 等效于FROM T1 ,T2 。它还等效于FROM T1 INNER JOIN T2 ON TRUE

```

## SQL的WHERE和HAVING子句

WHERE和HAVING的基本区别如下： WHERE在分组和聚集计算之前选取输入行(它控制哪些行进入聚集计算)， 而HAVING在分组和聚集之后选取输出行。 因此，WHERE子句不能包含聚集函数； 因为试图用聚集函数判断那些行将要输入给聚集运算是没有意义的。 相反，HAVING子句总是包含聚集函数。


## 视图

貌似经历的项目中视图运用的非常少,为什么?

## 事务

```
BEGIN
-- 语句
COMMIT | ROLLBACK;
```

## 窗口函数

窗口函数仅允许在查询的SELECT列表和ORDER BY子句中使用。 在其他地方禁止使用，比如GROUP BY,HAVING和WHERE子句。 这是因为它们逻辑上在处理这些子句之后执行。此外，窗口函数在标准聚合函数之后执行。 这意味在一个窗口函数的参数中包含一个标准聚合函数的调用是有效的，但反过来不行。

## 缺省值

- 如果没有明确声明缺省值，那么缺省值是 NULL 。
- 缺省值可以是一个表达式，它会在插入缺省值的时候计算(不是在创建表的时候)。
  `timestamp`字段可能有缺省值CURRENT_TIMESTAMP


## 约束

- 检查约束 CHECK
- 非空约束 NOT NULL
- 唯一约束 UNIQUE
- 主键 PRIMARY KEY
- 外键 REFERENCES
  ON DELETE RESTRICT
  ON DELETE CASCADE
- 排除约束 EXCLUDE

## 修改表

### 增加字段

```sql
ALTER TABLE table_name ADD COLUMN field_name data_type;
```

### 删除字段

```sql
ALTER TABLE table_name DROP COLUMN field_name CASCADE;
```

### 增加约束

```sql
ALTER TABLE table_name ADD CHECK (field_name <> '');
ALTER TABLE table_name ALTER COLUMN field_name SET NOT NULL;
```

### 删除约束

```sql
ALTER TABLE table_name DROP CONSTRAINT some_name;
ALTER TABLE table_name ALTER COLUMN field_name DROP NOT NULL;
```

### 改变字段的缺省值

```sql
ALTER TABLE table_name ALTER COLUMN field_name SET DEFAULT 0.00;
ALTER TABLE table_name ALTER COLUMN field_name DROP DEFAULT;
```

### 修改字段的数据类型

```sql
ALTER TABLE table_name ALTER COLUMN field_name TYPE numeric(10,2);
```

### 重命名字段

```sql
ALTER TABLE table_name RENAME COLUMN field_name TO field_new_name;
```

### 重命名表

```sql
ALTER TABLE table_name RENAME TO table_new_name;
```
