# Database Storage Part II

The DBMS assumes that the primary storage location of the database is on non-volatile disk.

The DBMS's components manage the movement of data between non-volatile and volatile storage.

## 1 Data Representation

A tuple is essentially a sequence of bytes. It's the job of the DBMS to interpret those bytes into attribute types and values.

The DBMS's catalogs contain the schema information about tables that the system uses to figure out the tuple's layout.

### 1.1 Data Types

INTEGER/BIGINT/SMALLINT/TINYINT

-   C/C++ Representation

FLOAT/REAL vs. NUMERIC/DECIMAL

-   IEEE-754 Standard / Fixed-point Decimals

VARCHAR/VARBINARY/TEXT/BLOB

-   Header with length, followed by data bytes.

TIME/DATE/TIMESTAMP

-   32/64-bit integer of (micro) seconds since Unix epoch.

#### 1.1.1 Variable Precision Numbers
Inexact, variable-precision numeric type that uses the "native" C/C++ types. Store directly as specified by IEEE-754.

Typically faster than arbitary precision numbers.

#### 1.1.2 Fixed Precision Numbers
Numeric data types with arbitary precision and scale. Used when round errors are unacceptable.

-   Example: NUMERIC, DECIMAL.

Typically stored in a exact, variable-length binary representation with additional meta-data.

-   Like a VARCHAR but not stored as a string.

#### 1.1.3 Large Values
Most DBMSs don't allow a tuple to exceed the size of a single page.

To store values that are larger than a page, the DBMS uses separate overflow storage pages.

#### 1.1.4 External Value Storage
Some systems allow you to store a really large value in an external file. Treated as a BLOB type.

The DBMS cannot manipulate the contents of an external file.

-   No durability protections.
-   No transaction protections.

## 2 System Catalogs

A DBMS stores meta-data about databases in its internal catalogs.

-   Table, columns, indexes, views
-   Users, permissions
-   Internal statistics

Almost every DBMS stores their databases' catalog in itself.

-   Wrap object abstraction around tuples.
-   Specialized code for "bootstrapping" catalog tables.

You can query the DBMS's internal INFORMATION<sub>SCHEMA</sub> catalog to get info about the database.

-   ANSI standard set of read-only views that provide info about all of the tables, views, columns, and procedures in a database.

## 3 Storage Models

### 3.1 Workloads

On-line Transaction Processing:

-   Simple queries that read/update a small amount of data that is related to a single entity in the database.

On-line Analytical Processing:

-   Complex queries that read large portions of the database spanning multiple entities.

The DBMS can store tuples in different ways that are better for either OLTP or OLAP workloads.

### 3.2 N-ary Storage Model (NSM)

Also known as row storage. The DBMS stores all attributes for a single tuple contiguously in a page.

Ideal for OLTP workloads where queries tend to operate only on an individual entity and insert-heavy workloads.

Advantages:

-   Fast inserts, updates, and deletes.
-   Good for queries that need the entire tuple.

Disadvantages:

-   Not good for scanning large portions of the table and/or a subset of the attributes.

### 3.3 Decomposition Storage Model (DSM)

Also known as column store. The DBMS stores the values of a single attribute for all tuples contiguously in a page.

Ideal for OLAP workloads where read-only queries perform large scans over a subset of the table's attributes.

Tuple Identifications:

-   Fixed-length Offsets. Each value is the same length for an attribute.
-   Embedded Tuple Ids. Each value is stored with its tuple id in a column.

Advantages:

-   Reduces the amount wasted I/O because the DBMS only reads the data it needs.
-   Better query processing and data compression.

Disadvantages:

-   Slow for point queries, inserts, updates, and deletes because of tuple splitting/stitching.
