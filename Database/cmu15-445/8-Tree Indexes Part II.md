# Tree Indexes Part II

## 1 Additional Index Usage

### 1.1 Implicit Indexes

Most DBMSs automatically create an index to enforce integrity constraints.

-   Primary Keys
-   Unique Constraints
-   Foreign Keys

### 1.2 Partial Indexes

Create an index on subset of entire table. This potentially reduces the size of indexes and the amount of overhead to maintain it.

One common use case is to partition indexes by date ranges.

-   Create a separate index per month, year.

    CREATE INDEX idx_foo
              ON foo (a, b)
           WHERE c = 'WuTang';


### 1.3 Covering Indexes

If all of the fields needed to process the query are available in an index, then the DBMS doesn't need to retrieve the whole tuple.

This reduces contention on DBMS's buffer pool resources.

    CREATE INDEX idx_foo
              ON foo (a, b);

    SELECT b FROM foo
     WHERE a = 123;



### 1.4 Index Include Columns

Embed addtional columns in indexes to support index-only queries. Not a part of search key. This stores addtional columns in leaf node.

    CREATE INDEX idx_foo
              ON foo (a, b)
         INCLUDE (c);


### 1.5 Functional/Expression Indexes

The index does not need to store keys in the same way that they appear in their base table.

You can use expressions when declaring an index.

    CREATE INDEX idx_user_login
        ON users (EXTRACT(dow FROM login));

    SELECT * FROM users
     WHERE EXTRACT(dow FROM login) = 2;



## 2 Skip Lists

Multiple levels of linked lists with extra pointers that skip over intermediate nodes. Maintain keys in sorted order without requiring global rebalancing.

A collection of lists at different levels

-   Lowest level is a sorted, single linked list of all keys.
-   2nd level links every other key.
-   3rd level links every fourth key.
-   In general, a level has half the keys of one below it.

To insert a key, flip a coin to decide how many levels to add the new key into. Provides approximately O(log n) search time complexity.

Advantages:

-   Uses less memory than a typical B+ Tree if you don't include reverse pointers.
-   Insertions and deletions do not require rebalancing.

Disadvantages:

-   Not disk/cache friendly because they do not optimize locality of references.
-   Reverse search is non-trivial.


## 3 Radix tree

Represent keys as individual digits. This allow threads to examine prefix one-by-one instead of comparing entire key.

-   The height of the tree depends on the length of the key.
-   Does not require rebalancing.
-   The path to a leaf node represents the key of the leaf.
-   Keys are stored implicitly and can be reconstructed from paths.


### 3.1 Radix Tree: Binary Comparable Keys

Not all attribute types can be composed into binary comparable digits for a radix tree.

-   Unsigned Integers: Byte order must be flipped for little endian machines.
-   Signed Integers: Flip two's-complement so that negative numbers are smaller than positive.
-   Floats: Classify into group (neg vs. pos, normalized vs. denormalized), then store as unsigned integer.
-   Compound: Transform each attribute separately.


## 4 Inverted Index

An inverted index stores a mapping of words to records that contain those words in the target attribute.

-   Sometimes called a full-text search index.
