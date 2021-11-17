# Tree Indexes Part I

## 1 Table Indexes

A table index is a replica of a subset of a table's columns that are organized and/or sorted for efficient access using a subset of those columns.

The DBMS ensures that the contents of the table and the index are logically in sync.

It's the DBMS's job to figure out the best index(es) to use to execute each query.

There is a trade-off on the number of indexes to create per database.

-   Storage Overhead
-   Maintenance Overhead

## 2 B+ Tree

A B+ Tree is a self-balancing tree data structure that keeps data sorted and allows searches, sequential access, insertions, and deletions in O(log n).

-   Generalization of a binary search tree in that a node can have more than two children.
-   Optimized for systems that read and write large blocks of data.

### 2.1 Properties

A B+tree is an M-way(M is maximum number of keys you can have in a single node)search tree with the following properties:

-   It is perfectly balanced (i.e., every leaf node is at the same depth).
-   Every inner node other than the root, is at least half-full M/2-1 <= #keys <= M-1
-   Every inner node with k keys has k+1 non-null children.

### 2.2 B+ Tree Nodes

Every node in the B+tree contains an array of key/value pairs.

-   The keys will always be the column or columns that you built your index on.
-   The values will differ based on whether the node is classified as inner nodes(pointer to another inner node or leaf node) or leaf nodes(record id or the tuple of contents).

The arrays are always kept in sorted order. When you load on a node, you do a binary search.

#### 2.2.1 Leaf Node Values
Approach #1: Record Ids

-   A pointer to the location of the tuple that the index entry corresponds to.

Approach #2: Tuple Data

-   The actual contents of the tuple is stored in the leaf node.
-   Secondary indexes have to store the record id as their values.
-   Typically it's used for primary key.

The size of B+tree node is usally the size of page. The sibling pointers are not pointing to memories, instead page ids. Because database need to go to the page table to fetch the page.

#### 2.2.2 Type
Clustered B+ tree

-   The sorted tuples are stored sequentially in the same page.

Unclustered B+ tree

-   The tuples in leaf nodes are distributed in different pages.

#### 2.2.3 B Tree VS. B+ Tree
The original B Tree from 1972 stored keys + values in all nodes in the tree.

-   More space efficient since each key only appears once in the tree.

A B+ Tree only stores values in leaf nodes. Inner nodes only guide the search process.

## 3 Operations

### 3.1 Insert

Find correct leaf L.

Put data entry into L in sorted order.

-   If L has enough space, done!
-   Else, must split L into L and a new node L2
    -   Redistribute entries evenly, copy up middle key.
    -   Insert index entry pointing to L2 into parent of L.

To split inner node, redistribute entries evenly, but push up middle key.

### 3.2 Delete

Start at root, find leaf L where entry belongs.

Remove the entry.

-   If L is at least half-full, done!
-   If L has only M/2-1 entries,
    -   Try to re-distribute, borrowing from sibling(adjacent node with same parent as L).
    -   If re-distribution fails, merge L and sibling.

If merge occurred, must delete entry(pointing to L or sibling) from parent of L.

### 3.3 B+ Tree Visualization

<https://cmudb.io/btree>

## 4 Clustered Indexes

The table is stored in the sort order specified by the primary key.

-   Can be either heap- or index-organized storage.

Some DBMSs always use a clustered index.

-   If a table doesn't include a pkey, the DBMS will automatically make a hidden row id key.

Other DBMSs cannot use them at all.

## 5 Selection Conditions

The DBMS can use a B+ Tree index if the query provides any of the attributes of the search key.

Example: Index on <a, b, c>

-   Supported: (a=5 AND b=3)
-   Supported: (b=3).

Not all DBMSs support this.

For hash index, we must have all attributes in search key.

## 6 B+ Tree Design Choices

### 6.1 Node Size

The slower the disk, the larger the optimal node size for B+ Tree.

-   HDD: ~1MB
-   SSD: ~10KB
-   In-Memory: ~512B

Optimal sizes can vary depending on the workload

-   Leaf Node Scans vs. Root-to-Leaf Traversals.

### 6.2 Merge Threshold

Some DBMSs don't always merge nodes when it is half full.

Delaying a merge operation may reduce the amount of reorganization.

May be better to just let underflows to exist and then periodically rebuild entire tree.

### 6.3 Variable Length Keys

Approach #1: Pointers

-   Store the keys as pointers to the tuple's attribute.

Approach #2: Variable Length Nodes

-   The size of each node in the B+ Tree can vary.
-   Requires careful memory management.

Approach #3: Key Map

-   Embed an array of pointers that map to the key + value list within the node.


### 6.4 Non-unique Indexes

Approach #1: Duplicate Keys

-   Use the same leaf node layout but store duplicate keys multiple times.

Approach #2: Value Lists

-   Store each key only once and maintain a linked list of unique values.

### 6.5 Intra-node Search

Approach #1: Linear

-   Scan node keys from beginning to end.

Approach #2: Binary

-   Jump to middle key, pivot left/right depending on comparsion.

Approach #3: Interpolation

-   Approaximate location of desired key based on known distribution of keys.

## 7 Optimizations

### 7.1 Prefix Compression

Stored keys in the same leaf node are likely to have the same prefix.

Instead of storing the entire key each time, extract common prefix and store only unique suffix for each key.

-   Many variations.

### 7.2 Suffix Truncation

The keys in the inner nodes are only used to "direct traffic".

-   We don't actually need the entire key.

Store a minimum prefix that is needed to correctly route probes into the index.

### 7.3 Bulk Insert

The fastest/best way to build a B+ Tree is to first sort the keys and then build the index from the bottom up.

### 7.4 Pointer Swizzling

Nodes use page ids to reference other nodes in the index. The DBMS has to get the memory location from the page table during travesal.

If a page is pinned in the buffer pool, then we can store raw pointers instead of page ids, thereby removing the need to get address from the page table.
