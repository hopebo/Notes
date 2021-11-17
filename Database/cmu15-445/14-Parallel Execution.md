# Parallel Execution

## 1 Parallel VS Distributed

Paralle DBMSs:

-   Nodes are physically close to each other.
-   Nodes connected with high-speed LAN.

Distributed DBMSs:

-   Nodes can be far from each other.
-   Nodes connected using public network.
-   Communication cost and problems cannot be ignored.


## 2 Process Model

A DBMS's process model defines how the system is architected to support concurrent requests from a multi-user  application.

A worker is the DBMS component that is responsible for executing tasks on behalf of the client and returning the results.

### 2.1 Approach #1: Process per DBMS Worker

Each worker is a separate OS process.

-   Relies on OS scheduler.
-   Use shared-memory for global data structures.
-   A process crash doesn't take down entire system.
-   Examples: IBM DB2, Postgres, Oracle.

### 2.2 Approach #2: Process Pool

A worker uses any process that is free in a pool.

-   Still relies on OS scheduler and shared memory.
-   Bad for CPU cache locality.
-   Examples: IBM DB2, Postgres(2015)

### 2.3 Approach #3: Thread per DBMS Worker

Single process with multiple worker threads.

-   DBMS has to manage its own scheduling.
-   May or may not use a dispatcher thread.
-   Thread crash (may) kill the entire system.
-   Examples: IBM DB2, MS SQL, MySQL, Oracle (2014)

-   Scheduling

    For each query plan, the DBMS has to decide where, when, and how to execute it.

    -   How many tasks should it use?
    -   How many CPU cores should it use?
    -   What CPU core should the tasks execute on?
    -   Where should a task store its output?

    The DBMS always knows more than operating system.

## 3 Inter- VS Intra-Query Parallelism

### 3.1 Inter-Query: Different queries are executed concurrently.

-   Increases throughput & reduces latency.

If queries are read-only, then this requires little coordination between queries.

If queries are updating the database at the same time, then this is hard to do this correctly.

-   Need to provide the illusion of isolation

### 3.2 Intra-Query: Execute the operations of a single query in parallel.

-   Decreases latency for long-running queries.

1.  Approach #1: Intra-Operator (Horizontal)

    Operators are decomposed into independent instances that perform the same function on different subsets of data.

    The DBMS inserts an exchange operator into the query plan to coalesce results from children operators.

2.  Approach #2: Inter-Operator (Vertical)

    Operations are overlapped in order to pipeline data from one stage to the next without materialization. Also called pipelined parallelism.


## 4 I/O Parallelism

Split the DBMS installation across multiple storage devices.
