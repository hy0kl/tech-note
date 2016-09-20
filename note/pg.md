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

