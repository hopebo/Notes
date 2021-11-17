# Join algorithms

We normalize tables in a relational database to avoid unnecessary repetition of information.

We use the join operate to reconstruct the original tuples without any information loss.

In general, we want the smaller table to always be the outer table.

## 1 Nested Loop Join

Summary:

-   Pick the smaller table as the outer table.
-   Buffer as much of the outer table in memory as possible.
-   Loop over the inner table or use an index.

### 1.1 Simple

    foreach tuple r in R: (outer table)
      foreach tuple s in S: (inner table)
        emit, if r and s match

The outer table has M pages and m tuples. The inner table has N pages and n tuples.

So the cost is: M + (m * N). In most cases, if we use the smaller table as the outer table, it will reduce the IO cost.


### 1.2 Block

Because the buffer pool reads tuples from the disk as pages. So we can do nested loop inside two pages first.

    foreach block Br in R:
      foreach block Bs in S:
        foreach tuple r in Br:
          foreach tuple s in Bs:
            emit, if r and s mathch

This algorithm performs fewer disk accesses.

-   For every block in R, it scans S once.

Cost: M + (M * N)

Also, the smaller table in terms of # of pages should be the outer table.

What if we have B buffers available?

-   Use B-2 buffers for scanning the outer table.
-   Use one buffer for the inner table, one buffer for sorting output.

This algorithm uses B-2 buffers for scanning R.

Cost: M + (M / (B - 2) * N)


### 1.3 Index

Use an index to find inner table matches to accelerate the join.

-   We could use an existing index for the join.
-   Or even build one on the fly.

    foreach tuple r in R:
      foreach tuple s in Index(ri = sj):
        emit, if r and s match

Assume the cost of each index probe is some constant C per tuple.

Cost: M + (m * C)

## 2 Sort-Merge Join

Phase #1: Sort

-   Sort both tables on the join key(s).
-   Can use the external merge sort algorithm that we talked about last clase.

Phase #2: Merge

-   Step through the two sorted tables in parallel, and emit matching tuples.
-   May need to backtrack depending on the join type.

Sort Cost (R): 2M &times; (log M / log B)

Sort Cost (S): 2N &times; (log N / log B)

Merge Cost: M + N

When is sort-merge join useful?

-   One or both tables are already sorted on join key.
-   Output must be sorted on join key.

The input relations may be sorted by either by an explicit sort operator, or by scanning the relation using an index on the join key.

## 3 Hash Join

Phase #1: Build

-   Scan the outer relation and populate a hash table using the hash function h1 on the join attributes.

Phase #2: Probe

-   Scan the inner relation and use h1 on each tuple to jump to a location in the hash table and find a matching tuple.


### 3.1 Hash table contents

Key: The attribute(s) that the query is joining the tables on.

Value: Varies per implementation.

-   Depends on what the operators above the join in the query plan expect as its output.

Approach #1: Full Tuple

-   Avoid having to retrieve the outer relation's tuple contents on a match.
-   Takes up more space in memory.

Approach #2: Tuple Identifier

-   Ideal for column stores because the DBMS doesn't fetch data from disk it doesn't need.
-   Also better if join selectivity is low.


## 4 Conclusion

Hashing is almost always better than sorting for operator execution.

Caveats:

-   Sorting is better on non-uniform data.
-   Sorting is better when result needs to be sorted.

Good DBMSs use either or both.
