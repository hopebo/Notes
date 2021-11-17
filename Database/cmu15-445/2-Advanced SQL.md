# Advanced SQL

## 1 SQL(sequel)

SQL is a collection of DML(Data Manipulation Language), DDL(Data Definition Language), DCL(Data Control Language).

-   DML

Commands like insert update delete select, the things that actually manipulate the data.

-   DDL

Create tables, define schemas.

-   DCL

Security authorization.

SQL is based on bags (duplicates) not sets (no duplicates).


## 2 Aggregates

Functions that return a single value from a bag(unordered, duplicates allowed) of tuples: Max, Min, Avg, Count.

Aggregate functions can only be used in the **SELECT** ouput list.

COUNT, SUM, AVG support **DISTINCT**.

Output of other columns outside of an aggregate is undefined. For example:

    SELECT AVG(s.gpa), e.cid
      FROM enrolled AS e, student AS s
     WHERE e.sid = s.sid

The above `e.cid` is undefined.

## 3 GROUP BY

Project tuples into subsets and calculate aggregates against each subset.

**Non-aggregated** values in SELECT output clause **must** appear in GROUP BY clause.

## 4 HAVING

Filters results based on aggregation computation. Like a WHERE clause for a GROUP BY.

## 5 String Operations

|         | String Case | String Quotes |
| ------- | ----------- | ------------- |
| Postgre | Sensitive   | Single Only   |
| MySQL   | Insensitive | Single/Double |


### 5.1 LIKE

LIKE is used for string matching. String-matching operators:

-   '%' Matches any substring (including empty strings).
-   '-' Match any one character.

## 6 DATE/TIME Operations

Operations to manipulate and modify DATE/TIME attributes.

Can be used in either output and predicates.

## 7 Output Redirection

Store query results in another table:

-   Table must not already be defined.
-   Table will have the same # of columns with the same types as the input.

Insert tuples from query into another table:

-   Inner SELECT must generate the same columns as the target table.
-   DBMSs have different options/syntax on what to do with duplicates.

## 8 Output Control

### 8.1 ORDER BY <column\*> [ASC|DESC]

Order the output tuples by the values in one or more of their columns.

### 8.2 Limit (Unsorted)

Limit the # of tuples returned in output.

Can set an offset to return a "range".

## 9 Nested Queries

Queries containing other queries. They are often difficult to optimize.

Inner queries can appear (almost) anywhere in query.

In this case:

    SELECT name FORM student
     WHERE sid IN (
       SELECT sid FROM enrolled
        WHERE cid = '15-445'
     )

For every single tuple in the outer query we execute the inner query over and over again.

## 10 Window Functions

Performs a calculation across a set of tuples that related to a single row. Like an aggregation but tuples are not grouped into a single output tuples.

    SELECT .. FUNC-NAME(...) OVER(...)
      FROM tableName

It's like combining the aggregation and the group by but in a single clause so the function is like the aggregation function and the over is like the group by.

-   Aggregation functions
-   Special window functions:
    -   ROW<sub>NUMBER</sub>() # of the current row.
    -   RANK() Order position of the current row.

The **OVER** keyword specifies how to group together tuples when computing the window function.

Use **PARTITION BY** to specify group.

## 11 Common Table Expressions(CTE)

Provides a way to write auxiliaxy statements for use in a larger query.

-   Think of it like a temp table just for one query.

Alternative to nested queries and views.

    WITH cteName (col1, col2) AS (
      SELECT 1, 2
    )
    SELECT col1 + col2 FROM cteName
