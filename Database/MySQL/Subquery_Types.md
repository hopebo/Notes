

# Types of subqueries in general

-   Single row subquery: returns zero or one row.
-   Multiple row subquery: returns one or more rows.
-   Multiple column subqueries: returns one or more columns.
-   Correlated subqueries: reference one or more columns in the outer SQL statement. The subquery is known as a correlated subquery because the subquery is related to the outer SQL statement.
-   Nested subqueries: subqueries are placed within another subquery.


# Different kinds of clauses in detail


## The subquery as scalar operand

In its simplest form, a subquery is a scalar subquery that returns a single value. A scalar subquery is a simple operand, and you can use it almost anywhere a single column value or literal is legal, and you can expect it to have those characteristics that all operands have: a data type, a length, an indication that it can be NULL, and so on.

    CREATE TABLE t1 (s1 INT);
    INSERT INTO t1 VALUES (1);
    CREATE TABLE t2 (s1 INT);
    INSERT INTO t2 VALUES (2);

    SELECT (SELECT s1 FROM t2) FROM t1;


## Subqueries within WHERE clause

The most common use of a subquery within WHERE clause is to compare the result of a subquery with `non_subquery_operand`.

The form of this is:

`non_subquery_operand comparison_operator (subquery)`

`comparison_operator` contains:  > < >= <= <> != <=> =

    SELECT * FROM t1
      WHERE column1 = (SELECT MAX(column2) FROM t2);

For a comparison of the subquery to a scalar, the subquery must return a scalar. For a comparison of the subquery to a row constructor, the subquery must be a row subquery that returns a row with the same number of values as the row constructor.

Scalar or column subqueries return a single value or a column of values. A row subquery is a subquery variant that returns a single row and can thus return more than one column value.

    SELECT column1,column2,column3
      FROM t1
      WHERE (column1,column2,column3) IN
             (SELECT column1,column2,column3 FROM t2);

If a subquery within WHERE clause returns multiple rows, the comparison against these results needs to be helped with **ANY IN ALL**.

    SELECT s1 FROM t1 WHERE s1 = ANY (SELECT s1 FROM t2);
    SELECT s1 FROM t1 WHERE s1 IN    (SELECT s1 FROM t2);
    SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);


## Derived table (inline view)


### FROM clause

A derived table is an expression that generates a table within the scope of a query FROM clause. For example, a subquery in a SELECT statement FROM clause is a derived table:
`SELECT ... FROM (subquery) [AS] tbl_name (col_list) ...`

    CREATE TABLE t1 (s1 INT, s2 CHAR(5), s3 FLOAT);

    INSERT INTO t1 VALUES (1,'1',1.0);
    INSERT INTO t1 VALUES (2,'2',2.0);
    SELECT sb1,sb2,sb3
      FROM (SELECT s1 AS sb1, s2 AS sb2, s3*2 AS sb3 FROM t1) AS sb
      WHERE sb1 > 1;


### JOIN clause

It’s fairly common to join a subquerywith the outer query rather than filtering in the WHERE clause.

    SELECT *
      FROM tutorial.sf_crime_incidents_2014_01 incidents
      JOIN ( SELECT date
               FROM tutorial.sf_crime_incidents_2014_01
              ORDER BY date
              LIMIT 5
           ) sub
        ON incidents.date = sub.date


### UNION clause

    SELECT COUNT(*) AS total_rows
      FROM (
            SELECT *
              FROM tutorial.crunchbase_investments_part1

             UNION ALL

            SELECT *
              FROM tutorial.crunchbase_investments_part2
           ) sub


## Semijoin

`SELECT FROM WHERE [NOT] EXISTS (subquery)`

The EXISTS operator checks if the row from the subquery matches any row in the outer query. If there’s no data matched, the EXISTS operator returns FALSE.

    SELECT DISTINCT store_type FROM stores
      WHERE EXISTS (SELECT * FROM cities_stores
                    WHERE cities_stores.store_type = stores.store_type);


## Correlated subquery

A correlated subquery is a subquery that contains a reference to a table that also appears in the outer query. A correlated subquery is evaluated once for each row. For example:

    SELECT * FROM t1
      WHERE column1 = ANY (SELECT column1 FROM t2
                           WHERE t2.column2 = t1.column2);


## View

View can be created with a subquery.

    CREATE VIEW cars_avgprice AS
      SELECT name, price
      FROM Cars
      WHERE price >
        (SELECT AVG(price) FROM Cars);


## Common table expression

To specify common table expressions, use a WITH clause that has one or more comma-separated subclauses. Each subclause provides a subquery that produces a result set, and associates a name with the subquery. The following example defines CTEs named cte1 and cte2 in the WITH clause, and refers to them in the top-level SELECT that follows the WITH clause:

    WITH
      cte1 AS (SELECT a, b FROM table1),
      cte2 AS (SELECT c, d FROM table2)
    SELECT b, d FROM cte1 JOIN cte2
    WHERE cte1.a = cte2.c;


# Optimizations on subqueries


## Semi-join

Semi-join can be applied to In (or =ANY) subqueries.
Phrase:

    SELECT ...
      FROM ot1, ...
        WHERE (oe1, ...) IN (SELECT ie1, ... FROM it1, ... WHERE ...);

Requirements:

1.  Subquery predicate is a deterministic IN/=ANY subquery predicate
2.  Subquery is a single SELECT (not a UNION)
3.  Subquery does not have GROUP BY
4.  Subquery does not use aggregate functions or HAVING
5.  Subquery does not use windowing functions
6.  Subquery predicate is (a) in an ON/WHERE clause, and (b) at the AND-top-level of that clause.
7.  Parent query block accepts semijoins (i.e we are not in a subquery of a single table UPDATE/DELETE
8.  We're not in a confluent table-less subquery, like "SELECT 1".
9.  Parent select is not a confluent table-less select
10. Neither parent nor child select have STRAIGHT_JOIN option.
11. No execution method was already chosen (by a prepared statement)

Different semi-join strategies are chosen based on cost:

1.  Convert the subquery to a join, or use table pullout and run the query as an inner join between subquery tables and outer tables. Table pullout pulls a table out from the subquery to the outer query.
2.  Duplicate Weedout: Run the semi-join as if it was a join and remove duplicate records using a temporary table.
3.  FirstMatch: When scanning the inner tables for row combinations and there are multiple instances of a given value group, choose one rather than returning them all. This "shortcuts" scanning and eliminates production of unnecessary rows.
4.  LooseScan: Scan a subquery table using an index that enables a single value to be chosen from each subquery's value group.
5.  Materialize the subquery into an indexed temporary table that is used to perform a join, where the index is used to remove duplicates. The index might also be used later for lookups when joining the temporary table with the outer tables; if not, the table is scanned.


## EXISTS strategy

EXISTS strategy can be applied to IN (or =ANY) subqueries and NOT IN(or <>ALL) subqueries.
Phrase:

    outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)

MySQL evaluates queries “from outside to inside.” That is, it first obtains the value of the outer expression outer_expr, and then runs the subquery and captures the rows that it produces.

A very useful optimization is to “inform” the subquery that the only rows of interest are those where the inner expression inner_expr is equal to outer_expr. This is done by pushing down an appropriate equality into the subquery's WHERE clause to make it more restrictive. The converted comparison looks like this:

    EXISTS (SELECT 1 FROM ... WHERE subquery_where AND outer_expr=inner_expr)


## Materialization

Materialization can be applied to IN (or =ANY) subqueries, NOT IN(or <>ALL) subqueries and derived tables.

Materialization can be applied to subquery predicates that appear anywhere (in the select list, WHERE, ON, GROUP BY, HAVING, or ORDER BY).

Requirements:

1.  Query must be an IN predicate.
2.  Subquery must be a single query specification clause (not a UNION).
3.  Subquery must have inner tables.
4.  The result of UNKNOWN (NULL) has the same meaning as a result of FALSE.


## Merge the derived table into the outer query block

This strategy can be only applied to derived tables.

The optimizer handles derived tables, view references, and common table expressions the same way: It avoids unnecessary materialization whenever possible, which enables pushing down conditions from the outer query to derived tables and produces more efficient execution plans.
