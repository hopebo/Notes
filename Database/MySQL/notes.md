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
