# Query Optimization

Remember that SQL is declarative.

-   User tells the DBMS what answer they want, not know how to get the answer.


## 1 Two approaches

Heuristics / Rules (written by human)

-   Rewrite the query to remove stupid / inefficient things.
-   Does not require a cost model.

Cost-based Search

-   Use a cost model to evalute multiple equivalent plans and pick the one with the lowest cost.

## 2 Query planning overview

SQL Query -> Parser -> (Abstract Syntax Tree) -> Binder (Name -> Internal ID) -> (Annotated AST) -> Rewriter -> Optimizer.

## 3 Relational algebra equivalences

Two relational algebra expressions are equivalent if they generate the same set of tuples.

The DBMS can identify better query plans without a cost model. This is often called **query rewriting**.

Selections:

-   Perform filters as early as possible.
-   Reorder predicates so that the DBMS applies the most selective one first.
-   Break a complex predicate, and push down.

Projections:

-   Perform them early to create smaller tuples and reduce intermediate results (if duplicates are eliminated).
-   Project out all attributes except the ones requested or required (e.g., joining keys).

This is not important for a column store.

Joins:

-   Commutative, associative.

## 4 Cost estimation

How long will a query take?

-   CPU: small cost; tough to estimate.
-   Disk: # of block transfers.
-   Memory: Amount of DRAM used.
-   Network: # of messages.

So we need to keep some statistics.

The DBMS stores internal statistics about the tables, attributes, and indexes in its internal catalog.

Different systems update them at different times.

Manual invocations:

-   Postgres/SQLite: ANALYZE
-   Oracle/MySQL: ANALYZE TABLE
-   SQL Server: UPDATE STATISTICS
-   DB2: RUNSTATS

### 4.1 Statistics

For each relation R, the DBMS maintains the following information:

-   Nr: Number of tuples in R.
-   V(A, R): Number of distinct values for attribute A.

The selection cardinality SC(A, R) is the average number of records with a value for an attribute A given Nr / V(A, R).

### 4.2 Complex predicates

The selectity(sel) of a predicate P is the fraction of tuples that qualify.

Formula depends on type of predicate:

-   Equality
-   Range
-   Negation
-   Conjunction
-   Disjunction

-   Equality

sel(A=constant) = SC(P) / V(A, R)

-  Range Query

sel(A>=a) = (Amax - a) / (Amax - Amin)

-  Negation Query

sel(not P) = 1 - sel(P)

-  Conjunction

sel(P1 &and; P2) = sel(P1) &times; sel(P2)

This assumes that the predicates are independent.

-  Disjunction

sel(P1 &or; P2) = sel(P1) + sel(P2) - sel(P1 &and; P2)

### 4.3 Result size estimation for joins

General case: Rcols &or; Scols = {A} where A is not a key for either table.

-   Match each R-tuple with S-tuples:
    estSize = Nr &times; Ns / V(A, S)
-   Symmetrically, for S:
    estSize = Nr &times; Ns / V(A, R)

Overall:
  estSize = Nr &times; Ns / max{V(A, S), V(A, R)}
