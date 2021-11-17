# Concurrency Control Theory

A DBMS's concurrency control and recovery components permeate throughout the design of its entire architecture.

## 1 Motivation

We both change the same record in a table at the same time.

How to avoid race condition?

You transfer $100 between bank accounts but there is a power failure.

What is the correct database state?

## 2 Transactions

A transaction is the execution of a sequence of one or more operations (e.g., SQL queries) on a shared database to perform some higher-level function.

It is the basic unit of change in a DBMS:

-   Partial transactions are not allowed!

## 3 Formal Definitions

Database: A fixed set of named data objects(e.g., A, B, C, &#x2026;)

-   We don't need to define what these objects are now.

Transaction: A sequence of read and write operations (R(A), W(B), &#x2026;)

-   DBMS's abtract view of a user program.

### 3.1 Transactions in SQL

A new txn starts with the BEGIN command.

The txn stops with either COMMIT or ABORT:

-   If commit, all changes are saved.
-   If abort, all changes are undone so that it's like as if the txn never executed at all.
-   Abort can be either self-inflicted or caused by the DBMS.

## 4 Correctness Criteria: ACID

Atomicity: All actions in the txn happen, or none happen. "all or nothing".

Consistency: If each txn is consistent and the DB starts consistent, then it ends up consistent. "it looks correct to me".

Isolation: Execution of one txn is isolated from that of other txns. "as if alone".

Durability: If a txn commits, its effect persist. "survive failures".

### 4.1 Mechanisms for Ensuring Atomicity

Approach #1: Logging

-   DBMS logs all actions so that it can undo the actions of aborted transactions.

Approach #2: Shadow Paging

-   DBMS makes copies of pages and txns make changes to those copies. Only when the txn commits is the page made visible to others.

### 4.2 Consistency

1.  Database Consistency

    The database accurately models the real world and follows integrity constraints.

    Transactions in the future see the effects of transactions committed in the past inside of the database.

2.  Transaction Consistency

    If the database is consistent before the transaction starts (running alone), it will also be consistent after.

    Transaction consistency is the application's responsibility.

### 4.3 Isolation of Transactions

Users submit txns, and each txn executes as if it was running by itself.

Concurrency is achieved by DBMS, which interleaves actions (reads/writes of DB objects) of various transactions.

1.  Mechanisms for Ensuring Isolation

    A concurrency control protocol is how the DBMS decides the proper interleaving of operations from multiple transactions.

    Two categories of protocols:

    -   Pessimistic: Don't let problems arise in the first place.
    -   Optimistic: Assume conflicts are rare, deal with them after they happen.

2.  Correctness

    How do we judge whether a schedule is correct?

    If the schedule is equivalent to some serial execution.

3.  Execution Schedule

    -   Serial Schedule
    -   Equivalent Schedules
    -   Serializable Schedule

4.  Conflicts

    -   Read-Write Conflicts ("Unrepeatable Reads")
    -   Write-Read Conflicts ("Dirty Reads")
    -   Write-Write Conflicts ("Lost Updates")

5.  Serializability

    -   Conflict Serializability
    -   View Serializability

### 4.4 Transaction Durability

All of the changes of committed transactions should be persistent.

-   No torn updates.
-   No changes from failed transactions.

The DBMS use either logging or shadow paging to ensure that all changes are durable.
