# Buffer Pools

Problem: How the DBMS manages its memory and move data back-and-forth from disk.

Spatial Control:

-   Where to write pages on disk.
-   The goal is to keep pages that are used together often as physically close together as possible on disk.

Temporal Control:

-   When to read pages into memory, and when to write them to disk.
-   The goal is minimize the number of stalls from having to read data from disk.

## 1 Buffer Pool Manager

### 1.1 Buffer Pool Organization

Memory region organized as an array of fixed-size pages. An array entry is called a frame.

When the DBMS requests a page, an exact copy is placed into one of these frames.

### 1.2 Buffer Pool Meta-data

The page table keeps track of pages that are currently in the memory.

Also maintains additional meta-data per page:

-   Dirty Flag
-   Pin/Reference Counter

### 1.3 Locks VS. Latches

Locks:

-   Protects the database's logical contents from other transactions.
-   Held for transaction duration.
-   Need to be able to rollback changes.

Latches (Mutex):

-   Protects the critical sections of the DBMS's internal data structure from other threads.
-   Held for operation duration.
-   Do not need to be able to rollback changes.

### 1.4 Page Table VS. Page Directory

The page directory is the mapping from page ids to page locations in the database files.

-   All changes must be recorded on disk to allow the DBMS to find on restart.

The page table is the mapping from page ids to a copy of the page in buffer pool frames.

-   This is an in-memory data structure that does not need to be stored on disk.

### 1.5 Multiple Buffer Pools

The DBMS does not always have a single buffer pool for the entire system.

-   Multiple buffer pool instances
-   Per-database buffer pool
-   Per-page type buffer pool

Helps reduce latch contention and improve locality.

### 1.6 Pre-fetching and Scan Sharing

#### 1.6.1 Pre-fetching
The DBMS can also prefetch pages based on a query plan.

-   Sequential Scans
-   Index Scans

#### 1.6.2 Scan Sharing
Queries are able to reuse data retrieved from storage or operator computations.

-   This is different from result caching.

Allow multiple queries to attach to a single cursor that scans a table.

-   Queries do not have to be exactly the same.
-   Can also share intermediate results.

If a query starts a scan and if there one already doing this, then the DBMS will attach to the second query's cursor.

-   The DBMS keeps track of where the second query joined with the first so that it can finish the scan when it reaches the end of the data structure.

### 1.7 OS Page Cache

Most disk operations go through the OS API. Unless you tell it not to, the OS maintains its own filesystem cache.

Most DBMSs use direct I/O (O<sub>DIRECT</sub>) to bypass the OS's cache.

-   Redundant copies of pages.
-   Different eviction policies.

## 2 Buffer Replacement Policies

When the DBMS needs to free up a frame to make room for a new page, it must decide which page to evict from the buffer pool.

Goals:

-   Correctness
-   Accuracy
-   Speed
-   Meta-data overhead

### 2.1 Least Recently Used

Maintain a timestamp of when each page was last accessed. When the DBMS needs to evict a page, select the one with oldest timestamp.

-   Keep the pages in sorted order to reduce the search time on eviction.

### 2.2 Clock

Approximation of LRU without needing a separate timestamp per page.

-   Each page has a reference bit.
-   When a page is accessed, set to 1.

Organize the pages in a circular buffer with a "clock hand":

-   Upon sweeping, check if a page's bit is set to 1.
-   If yes, set to zero. If no, then evict.

Problems:

LRU and Clock replacement policies are susceptible to sequential flooding.

-   A query performs a sequential scan that reads every page.
-   This pollutes the buffer pool with pages that are read once and then never again.

The most recently used page is actually the most unneeded page.

### 2.3 Better Policies: LRU-K

Take into account history of the last K references as timestamps and compute the interval between subsequent accesses.

The DBMS then uses this history to estimate the next time that page is going to be accessed.

### 2.4 Better Policies: Localization

The DBMS chooses which pages to evict on a per txn/query basis. This minimizes the pollution of the buffer pool from each query.

-   Keep track of the pages that a query has accessed.

### 2.5 Better Policies: Priority Hints

The DBMS knows what the context of each page during query execution.

It can provide hints to the buffer pool on whether a page is important or not.

## 3 Dirty Pages

FAST: If a page in the buffer pool is not dirty, then the DBMS can simply "drop" it.

SLOW: If a page is dirty, then the DBMS must write back to disk to ensure that its changes are persisted.

Trade-off between fast evictions versus dirty writing pages that will not be read again in the future.

### 3.1 Background Writing

The DBMS can periodically walk through the page table and write dirty pages to disk.

When a dirty page is safely written, the DBMS can either evict the page or just unset the dirty flag.

Need to be careful that we don't write dirty pages before their log records have been written.

## 4 Allocation Policies

Global Policies:

-   Make decisions for all active txns.

Local Policies:

-   Allocate frames to a specific txn without considering the behavior of concurrent txns.
-   Still need to support sharing pages.

## 5 Other Memory Pools

The DBMS needs memory for things other than just tuples and indexes.

These other memory pools may not always backed by disk. Depends on implementation.

-   Sorting + Join Buffers
-   Query Caches
-   Maintenance Buffers
-   Log Buffers
-   Dictionary Caches
