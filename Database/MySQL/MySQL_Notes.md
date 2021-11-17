## Terminology
- Index Condition Pushdown (ICP)
Index Condition Pushdown (ICP)是MySQL用索引去表里取数据的一种优化。如果禁用ICP，引擎层会通过索引在基表中寻找数据行，然后返回给MySQL Server层，再去为这些数据行进行WHERE后的条件的过滤。ICP启用，如果部分WHERE条件能使用索引中的字段，MySQL Server会把这部分下推到引擎层。存储引擎通过使用索引条目，然后推索引条件进行评估，使用这个索引把满足的行从表中读取出。ICP能减少引擎层访问基表的次数和MySQL Server 访问存储引擎的次数。

## Window Function
### Definition
A window function performs an aggregate-like operation on a set of query rows. However, whereas an aggregate operation groups query rows into a single result row, a window function produces a result for each query row:
- The row for which function evaluation occurs is called the current row.
- The query rows related to the current row over which function evaluation occurs comprise the window for the current row.

Window 函数的计算发生在 GROUP BY 之后，如果 PARTITION BY 的列是 GROUP BY 的子集，那么与 sql_mode = only\_full\_group\_by 的模式兼容。否则不兼容。

Frame 是 Window 窗口包含数据列的子集，可以按照行号或者范围进行选取，提供更小粒度的分析。

## Insoure Build
在 Cmake 命令行选项中加上 -DFORCE\_INSOURCE\_BUILD=1

## 生成随机日期
```sql
select concat('2020', mod(floor(1 + rand() * 10), 9), '-',floor(1 + rand() * 28),' ', floor(8 + rand() * 10), ':', floor(10 + rand() * 49), ':', floor(10 + rand() * 49));
```

## 生成随机数
```sql
select FLOOR(RAND() * 10);
```

## PSI代表Performance schema instrumentation

## 创建用户并授权
```sql
CREATE USER 'test' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT ALL PRIVILEGES ON *.* to 'test'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

## 连接远程库命令
`mysql -h 10.125.193.79 -u myadmin -pPassw0rd -P 3308  -D mysql_parallet_query_sql_collections_test`

## 使用federated引擎创建远程表
`CREATE TABLE IF NOT EXISTS remote_t1(a INT) ENGINE=federated connection="mysql://remote_user:123456@11.160.45.141:33941/test/t1";`

## 从网络包获取开始的断点
`b dispatch_command`

## 解析SQL
`b mysql_parse`

## 执行SQL
```
mysql_execute_command()
  |- Sql_cmd_dml::execute()
       |- Sql_cmd_dml::execute_inner()
          |- SELECT_LEX::optimize()
```

## federated引擎执行
在handler进行Init()过程中如果连接失效会进行重连，否则直接使用。使用MYSQL client接口发送查询SQL，获得全部的查询结果。
在ha\_rnd\_next()过程中，每次循环就会往后迭代一次查询结果，并将结果集转换为handler内部的格式。

不会根据优化后的结果改进SQL，只会尝试查询返回全部结果，在连接数据库的时候就已经确定了select query string。

## 在SQL中强制使用INDEX
- 不使用某个表的索引
`select /*+ NO_POLARDB_INDEX(t2) */ *  from t2 where id = 1;`

- 使用某个表的某个索引
`select /*+ POLARDB_INDEX(t1 i1) */ * from t1 where c1 > 0;`

- 社区使用INDEX
```
SELECT * FROM v1 USE INDEX (PRIMARY) WHERE c1=2;
SELECT * FROM v1 FORCE INDEX (PRIMARY) WHERE c1=2;
SELECT * FROM v1 IGNORE INDEX (PRIMARY) WHERE c1=2;
```

## EXPLAIN Join Types
The type column of EXPLAIN output describes how tables are joined. In JSON-formatted output, these are found as values of the access_type property. The following list describes the join types, ordered from the best type to the worst:

- system
The table has only one row (= system table). This is a special case of the const join type.

- const
The table has at most one matching row, which is read at the start of the query. Because there is only one row, values from the column in this row can be regarded as constants by the rest of the optimizer. const tables are very fast because they are read only once.

const is used when you compare all parts of a PRIMARY KEY or UNIQUE index to constant values. In the following queries, tbl_name can be used as a const table:

```sql
SELECT * FROM tbl_name WHERE primary_key=1;

SELECT * FROM tbl_name
  WHERE primary_key_part1=1 AND primary_key_part2=2;
```

- eq_ref
One row is read from this table for each combination of rows from the previous tables. Other than the system and const types, this is the best possible join type. It is used when all parts of an index are used by the join and the index is a PRIMARY KEY or UNIQUE NOT NULL index.

eq\_ref can be used for indexed columns that are compared using the = operator. The comparison value can be a constant or an expression that uses columns from tables that are read before this table. In the following examples, MySQL can use an eq\_ref join to process ref\_table:

```sql
SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column=other_table.column;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column_part1=other_table.column
  AND ref_table.key_column_part2=1;
```

- ref
All rows with matching index values are read from this table for each combination of rows from the previous tables. ref is used if the join uses only a leftmost prefix of the key or if the key is not a PRIMARY KEY or UNIQUE index (in other words, if the join cannot select a single row based on the key value). If the key that is used matches only a few rows, this is a good join type.

ref can be used for indexed columns that are compared using the = or <=> operator. In the following examples, MySQL can use a ref join to process ref_table:

```sql
SELECT * FROM ref_table WHERE key_column=expr;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column=other_table.column;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column_part1=other_table.column
  AND ref_table.key_column_part2=1;
```

- fulltext
The join is performed using a FULLTEXT index.

- ref\_or\_null
This join type is like ref, but with the addition that MySQL does an extra search for rows that contain NULL values. This join type optimization is used most often in resolving subqueries. In the following examples, MySQL can use a ref\_or\_null join to process ref\_table:

```sql
SELECT * FROM ref_table
  WHERE key_column=expr OR key_column IS NULL;
```

- index\_merge
This join type indicates that the Index Merge optimization is used. In this case, the key column in the output row contains a list of indexes used, and key_len contains a list of the longest key parts for the indexes used.

- unique\_subquery
This type replaces eq_ref for some IN subqueries of the following form:

```sql
value IN (SELECT primary_key FROM single_table WHERE some_expr)
```

unique_subquery is just an index lookup function that replaces the subquery completely for better efficiency.

- index\_subquery
This join type is similar to unique\_subquery. It replaces IN subqueries, but it works for nonunique indexes in subqueries of the following form:

```sql
value IN (SELECT key_column FROM single_table WHERE some_expr)
```

- range
Only rows that are in a given range are retrieved, using an index to select the rows. The key column in the output row indicates which index is used. The key_len contains the longest key part that was used. The ref column is NULL for this type.

range can be used when a key column is compared to a constant using any of the =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, LIKE, or IN() operators:

```sql
SELECT * FROM tbl_name
  WHERE key_column = 10;

SELECT * FROM tbl_name
  WHERE key_column BETWEEN 10 and 20;

SELECT * FROM tbl_name
  WHERE key_column IN (10,20,30);

SELECT * FROM tbl_name
  WHERE key_part1 = 10 AND key_part2 IN (10,20,30);
```

- index
The index join type is the same as ALL, except that the index tree is scanned. This occurs two ways:

If the index is a covering index for the queries and can be used to satisfy all data required from the table, only the index tree is scanned. In this case, the Extra column says Using index. An index-only scan usually is faster than ALL because the size of the index usually is smaller than the table data.

 - A full table scan is performed using reads from the index to look up data rows in index order. Uses index does not appear in the Extra column.

 - MySQL can use this join type when the query uses only columns that are part of a single index.

- ALL
A full table scan is done for each combination of rows from the previous tables. This is normally not good if the table is the first table not marked const, and usually very bad in all other cases. Normally, you can avoid ALL by adding indexes that enable row retrieval from the table based on constant values or column values from earlier tables.

Refer to: [explain-join-types](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-join-types).
