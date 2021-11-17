# Database Storage Part I

Problem: How the DBMS represents the database in files on disk.

**Query Planning** -> **Operator Execution** -> **Access Methods** -> **Buffer Pool Manager** -> **Disk Manager**

## 1 Access Times

| Device          | Time(ns)      |
| --------------- | ------------- |
| L1 Cache Ref    | 0.5           |
| L2 Cache Ref    | 7             |
| DRAM            | 100           |
| SSD             | 150,000       |
| HDD             | 10,000,000    |
| Network Storage | ~30,000,000   |
| Tape Archives   | 1,000,000,000 |

## 2 Why not use the OS (mmp)?

DBMS (almost) always wants to control things itself and can do a better job at it.

-   Flushing dirty pages to disk in the correct order.
-   Specialized prefetching.
-   Buffer replacement policy.
-   Thread / process scheduling.

## 3 Database Pages

A page is a fixed-size block of data.

-   It can contain tuples, meta-data, indexes, log records...;
-   Most systems do not mix page types.
-   Some systems require a page to be self-contained.

Each page is given a unique identifier.

-   The DBMS uses an indirection layer to map page ids to physical locations.

There are three different notions of "pages" in a DBMS:

-   Hardware Page (usually 4KB)
-   OS Page (usually 4KB)
-   Database Page (1-16KB)

By hardware page, we mean at what level the device can guarantee a "failsafe write".

| DBMS                   | Database Page(KB) |
| ---------------------- | ----------------- |
| SQLite                 | 1                 |
| DB2, ORACLE            | 4                 |
| SQL Server, PostgreSQL | 8                 |
| MySQL                  | 16                |

## 4 File Storage and Storage Manager

The DBMS stores a database as one or more files on disk.

The storage manager is responsible for maintaining a database's files. It organizes the files as a collection of pages.

-   Tracks data read/written to pages.
-   Tracks the available space.

## 5 Page File Architecture

### 5.1 Heap File

A heap file is an unordered collection of pages where tuples that are stored in random order.

-   Get / Delete page.
-   Must also support iterating over all pages.

Need meta-data to keep track of what pages exist and which ones have free space.

Two ways to represent a heap file:

-   Linked List
-   Page Directory

#### 5.1.1 Heap File: Linked List
Maintain a header page at the beginning of the file that stores two pointers:

-   HEAD of the free page list.
-   HEAD of the data page list.

Each page keeps track of the number of free slots in itself.

#### 5.1.2 Heap File: Page Directory
The DBMS maintains special pages that tracks the location of data pages in the database files.

The directory also records the number of free slots per page.

The DBMS has to make sure the directory pages are in sync with the data pages.


### 5.2 Page Layout

#### 5.2.1 Page Header
Every page contains a header of meta-data about the page's contents.

-   Page Size
-   Checksum
-   DBMS Version
-   Transaction Visibility
-   Compression Information

#### 5.2.2 Page Layout

##### 5.2.2.1 Tuple-oriented
The most common layout scheme is called slotted pages. The slot array maps "slots" to the tuples' starting position offsets.

The header keeps track of:

-   The # of used slots
-   The offset of the starting location of the last slot used.

##### 5.2.2.2 Log-structured
Instead of storing tuples in pages, the DBMS only stores log records.

The system appends log records to the file of how the database was modified:

-   Inserts store the entire tuple.
-   Deletes mark the tuple as deleted.
-   Updates contain the delta of just the attributes that were modified.

To read a record, the DBMS scans the log backwards and "recreates" the tuple to find what it needs.

Build indexes to allow it to jump to locations in the log.

Periodically compacting the logs coalesces larger log files into smaller files by removing unnecessary records.

### 5.3 Tuple Layout

A tuple is essentially a sequence of bytes. It's the job of DBMS to interpret those bytes into attribute type and values.

Each tuple is prefixed with a header that contains meta-data about it.

-   Visibility info (concurrency control)
-   Bit Map for NULL values.

Attributes are typically stored in the order that you specify them when you create the table.

Can physically denormalize (e.g., "pre-join") related tuples and store them together in the same page.

-   Potentially reduces the amount of I/O for common workload patterns.
-   Can make updates more expensive.

#### 5.3.1 Record IDs
The DBMS needs a way to keep track of individual tuples.

Each tuple is assigned a unique record identifier.

-   Most common: page<sub>id</sub> + offset/slot.
-   Can also contain file location info.
