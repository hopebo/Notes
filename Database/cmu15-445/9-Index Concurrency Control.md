# Index Concurrency Control

A concurrency control protocol is the method that the DBMS uses to ensure "correct" results for concurrent operations on a shard object.

## 1 Locks VS. Latches

Locks

-   Protects the index's logical contents from other txns.
-   Held for txn duration.
-   Need to be able to rollback changes.

Latches

-   Protects the critical sections of the index's internal data structure from other threads.
-   Held for operation duration.
-   Do not need to be able to rollback changes.

| Locks    | Latches                              |                           |
| -------- | ------------------------------------ | ------------------------- |
| Separate | User transaction                     | Threads                   |
| Protect  | Database Contents                    | In-Memory Data Structures |
| During   | Entire Transactions                  | Critical Sections         |
| Modes    | Shared, Exclusive, Update, Intention | Read, Write               |
| Deadlock | Detection & Resolution               | Avoidance                 |
| by       | Waits-for, Timeout, Aborts           | Coding Discipline         |
| Kept in  | Lock Manager                         | Protected Data Structure  |


Read Mode

-   Multiple threads are allowed to read the same item at the same time.
-   A thread can acquire the read latch if another thread has it in read mode.

Write Mode

-   Only one thread is allowed to access the item.
-   A thread cannot acquire a write latch is another thread holds the latch in any mode.

## 2 B+ Tree Concurrency Control

We want to allow multiple threads to read and update a B+ tree at the same time.

We need to protect from two types of problems:

-   Threads trying to modify the contents of a node at the same time.
-   One thread traversing the tree while another thread splits/merges nodes.


### 2.1 Latch Crabbing/Coupling

Protocol to allow multiple threads to access/modify B+ Tree at the same time.

Basic Idea:

-   Get latch for parent.
-   Get latch for child.
-   Release latch for parent if "safe".

A safe node is one that will not split or merge when updated.

-   Not full (on insertion)
-   More than half-full (on deletion)

Search: Start at root and go down; repeatedly,

-   Acquire R latch on child
-   Then unlatch parent

Insert/Delete: Start at root and go down, obtaining W latches as needed. Once child is latched, check if it is safe:

-   If child is safe, release all latches on ancestors.


### 2.2 Better Latching Algorithm

Assume that the leaf node is safe.

Use read latches and crabbing to reach it, and verify that it is safe.

If leaf is not safe, then do previous algorithm using write latches.

Search: Same as before.

Insert/Delete:

-   Set latches as if for search, get to leaf, and set W latch on leaf.
-   If leaf is not safe, release all latches, and restart thread using the previous insert/delete protocol with write latches.

This approach optimistically assumes that only leaf node will be modified; if not, R latches set on the first pass to leaf are wasteful.


### 2.3 Leaf Node Scans

Latches do not support deadlock detection or avoidance. The only way we can deal with this problem is through coding discipline.

The leaf node sibling latch acquisition protocol must support a "no-wait" mode. B+ tree code must cope with failed latch acquisitions.


### 2.4 Delayed Parent Updates

Every time a leaf node overflows, we have to update at least three nodes.

-   The leaf node being split.
-   The new leaf node being created.
-   The parent node.

Blink Tree Optimization: When a leaf node overflows, delay updating its parent node.
