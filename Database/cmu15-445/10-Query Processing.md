# Query Processing

## 1 Query Plan

The operators are arranged in a tree. Data flows from the leaves toward the root.

The ouput of the root node is the result of the query.

## 2 Processing Model

A DBMS's processing model defines how the system executes a query plan.

-   Different trade-offs for different workloads.

Three approaches:

-   Iterator model
-   Materialization model
-   Vectorized / Batch model

### 2.1 Iterator Model

Each query plan operator implements a next function.

-   On each invocation, the operator returns either a single tuple or a null marker if there are no more tuples.
-   The operator implements a loop that calls next on its children to retrieve their tuples and then process them.

Top-down plan processing. Also called Volcano or Pipeline Model.

This is used in almost every DBMS. Allows for tuple pipelining.

Some operators will block until children emit all of their tuples.

-   Joins, Subqueries, Order by.

Output control works easily with this approach.

-   Limit.


### 2.2 Materialization Model

Each operator processes its input all at once and then emits its output all at once.

-   The operator "materializes" its output as a single result.
-   The DBMS can push down hints into to avoid scanning too many tuples.

Bottom-up plan processing.

Better for OLTP workloads because queries typically only access a small number of tuples at a time.

-   Lower execution / coordination overhead.

Not good for OLAP queries with large intermediate results.


### 2.3 Vectorization Model

Like iterator model, each operator implements a next function.

Each operator emits a batch of tuples instead of a single tuple.

-   The operator's internal loop processes multiple tuples at a time.
-   The size of batch can vary based on hardware or query properties.

Ideal for OLAP queries

-   Greatly reduces the number of invocations per operator.
-   Allows for operators to use vectorized (SIMD) instructions to process batches of tuples.

### 2.4 Summary

| Iterator/Volcano | Vectorized      | Materialization |                  |
| ---------------- | --------------- | --------------- | ---------------- |
| Direction        | Top-Down        | Top-Down        | Bottom-Up        |
| Emits            | Single Tuple    | Tuple Batch     | Entire Tuple Set |
| Target           | General Purpose | OLAP            | OLTP             |

## 3 Access Methods

An access method is a way that the DBMS can access the data stored in a table.

-   Not defined in relational algebra.

Three basic approaches:

-   Sequential scan.
-   Index scan.
-   Multi-index / "Bitmap" scan.


### 3.1 Sequential Scan

For each page in the table:

-   Retrieve it from the buffer pool.
-   Iterate over each tuple and check whether to include it.

The DBMS maintains an internal cursor that tracks the last page / slot it examined.

This is almost the worst thing that the DBMS can do to execute a query.

Sequential scan optimizations:

-   Prefetching
-   Parallelization
-   Buffer pool bypass
-   Zone maps
-   Late materialization
-   Heap clustering

#### 3.1.1 Zone Maps
Pre-computed aggregates for the attribute values in a page. DBMS checks the zone map first to decide whether it wants to access the page.

#### 3.1.2 Late Materialization
DSM DBMSs can delay stitching together tuples until the upper parts of the query plan.

#### 3.1.3 Heap Clustering
Tuples are sorted in the heap's pages using the order specified by a clustering index.

If the query accesses tuples using the clustering index's attributes, then the DBMS can jump directly to the pages that it needs.

### 3.2 Index Scan

The DBMS picks an index to find the tuples that the query needs.

Which index to use depends on:

-   What attributes the index contains
-   What attributes the query references
-   The attribute's value domains
-   Predicate composition
-   Whether the index has unique or non-unique keys


### 3.3 Multi-index Scan

If there are multiple indexes that the DBMS can use for a query:

-   Compute sets of record ids using each matching index.
-   Combine these sets based on the query's predicates (union vs. intersect).
-   Retrieve the records and apply any remaining terms.

Set intersection can be done with bitmaps, hash tables, or Bloom filters.


## 4 Index Scan Page Sorting

Retrieving tuples in the order that appear in an unclustered index is inefficient.

The DBMS can first figure out all the tuples that it needs and then sort them based on their page id.

## 5 Expression evaluation

The DBMS represents a WHERE clause as an expression tree.

The nodes in the tree represent different expression types:

-   Comparisons (= , < , > , !=)
-   Conjunction (AND), Disjunction (OR)
-   Arithmetic Operators (+, -, \*, /, %)
-   Constant Values
-   Tuple Attribute References

To evaluate an expression tree at runtime, the DBMS maintains a context handle that contains metadata for the execution, such as the current tuple, the parameters, the parameters, and the table schema. The DBMS then walks the tree to evaluate its operators and produce a result.
