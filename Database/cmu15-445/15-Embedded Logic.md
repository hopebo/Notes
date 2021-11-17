# Embedded Logic

## 1 User-defined functions

A user-defined function (UDF) is a function written by the application developer that extends the system's functionality beyond its built-in operations.

-   It takes in input arguments (scalars)
-   Perform some computation
-   Return a result (scalars, tables)

## 2 Stored procedures

A stored procedure is a self-contained function that performs more complex logic inside of the DBMS.

-   Can have many input/output parameters.
-   Can modify the database table/structures.
-   Not normally used within a SQL query.

## 3 Stored procedures VS UDF

A UDF is meant to perform a subset of a read-only computation within a query.

A stored procedure is meant to perform a complete computation that is independent of query.

## 4 Database triggers

A trigger instructs the DBMS to invoke a UDF when some event occurs in the database.

The developer has to define:

-   What type of event will cause it to fire.
-   The scope of the event.
-   When it fires relative to that event.

Event type:

-   INSERT
-   UPDATE
-   DELETE
-   TRUNCATE
-   CREATE
-   ALTER
-   DROP

Event scope:

-   TABLE
-   DATABASE
-   VIEW
-   SYSTEM

Trigger timing:

-   Before the statement executes.
-   After the statement executes.
-   Before each row that the statement affects.
-   After each row that the statement affects.
-   Instead of the statement.


## 5 Change notifications

A change notification is like a trigger except that the DBMS sends a message to an external entity that something notable has happened in the database.

-   Think a "pub/sub" system.
-   Can be chained with a trigger to pass along whenever a change occurs.

SQL standard: LISTEN + NOTIFY.


## 6 Complex types

Approach #1: Attribute splitting

-   Store each primitive element in the complex type as its own attribute in the table.

Approach #2: Application serialization

-   Java serialize, Python pickle
-   Google protobuf, Facebook thrift
-   JSON / XML


## 7 User-defined types

A user-defined type is a special data type that is defined by the application developer that the DBMS can stored natively.


## 8 Views

Create a "virtual" table containing the output from a SELECT query. The view can then be accessed as if it was a real table.

This allows programmers to simplify a complex query that is executed often.

-   Won't make it faster though.

Often used as mechanism for hiding a subset of a table's attributes from certain users. The DBMS will rewrite the SQL.

## 9 Views VS Select Into

VIEW

-   Dynamic results are only materialized when needed.

SELECT&#x2026;INTO

-   Creates static table that does not get updated when student gets updated.

MATERIALIZED VIEWS
Create a view containing the output from a SELECT query that is automatically updated when the underlying tables change.
