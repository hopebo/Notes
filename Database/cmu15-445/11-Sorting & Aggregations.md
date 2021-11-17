# Sorting & Aggregations

Why do we need to sort?

Tuples in a table have no specific order.

But users often want to retrieve tuples in a specific order.

-   Trivial to support duplicate elimination (DISTINCT)
-   Bulk loading sorted tuples into a B+ tree index is faster
-   Aggregations (GROUP BY)

## 1 Sorting Algorithms

### 1.1 External Merge Sort

Sorting Phase

-   Sort small chunks of data that fit in main-memory, and then write back the sorted data to a file on disk.

Merge Phase

-   Combine sorted sub-files into a single larger file.

Files are broken up into N pages. The DBMS has a finite number of B fixed-size buffers.

#### 1.1.1 Two-way External Merge Sort
Pass #0

-   Read every B pages of the table into memory.
-   Sorts them, and write them back to disk.
-   Each sorted set of pages is called a run.

Pass #1,2,3,...;

-   Recursively merges pairs of runs into runs twice as long.
-   Use three buffer pages (2 for input pages, 1 for output).

#### 1.1.2 General External Merge Sort
Pass #0

-   Use B buffer pages.
-   Produce ceiling(N/B) sorted runs of size B

Pass #1,2,3,...;

-   Merge B-1 runs (i.e. K-way merge)

Number of passes = 1 + logB-1(ceiling(N/B))

a<sub>1</sub>

-   Use B+ tree
    -   Good for clustered B+ tree
    -   For unclustered B+ tree, this is almost always a bad idea. In general, one I/O per data record. The DBMS will choose the best one for us. Whether to use index, or do external merge sort.


## 2 Aggregation

### 2.1 Sort

### 2.2 Hashing

Hashing is a better alternative in this scenario.

-   Only need to remove duplicates, no need for ordering.
-   Can be computationally cheaper than sorting.
