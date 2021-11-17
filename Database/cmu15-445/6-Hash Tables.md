# Hash Tables

Design Decisions:

Data Organization

-   How we layout data structure in memory/pages and what information to store to support efficient access.

Concurrency

-   How to enable multiple threads to access the data structure at the same time without causing problems.

A hash table implements an associative array abstract data type that maps keys to values.

It use a hash function to compute an offset into the array, from which the desired value can be found.

## 1 Hash Table

Design Decision #1: Hash Function

-   How to map a large key space into a smaller domain.
-   Trade-off between being fast vs. collision rate.

Design Decision #2: Hashing Scheme

-   How to handle key collision after hashing.
-   Trade-off between allocating a large hash table vs. additional instructions to find/insert keys.

## 2 Hash Functions

MurmurHash(2008)

-   Designed to a fast, general purpose hash function.

Google CityHash(2011)

-   Based on ideas from MurmurHash2
-   Designed to be faster for short keys (<64 bytes).

Google FarmHash(2014)

-   Newer version of CityHash with better collision rates.

CLHash(2016)

-   Fast hashing function based on carry-less multiplication.

## 3 Hash Schemes

### 3.1 Static Hashing Schemes

#### 3.1.1 Linear Probe Hasing
Single gaint table of slots.

Resolve collisions by linearly searching for the next free slot in the table.

-   To determine whether an element is present, hash to a location in the index and scan for it.
-   Have to store the key in the index to know when to stop scanning.
-   Insertions and deletions are generalizations of lookups.

##### 3.1.1.1 Non-unique Keys
Choice #1: Separate Linked List

-   Store values in separate storage area for each key.

Choice #2: Redundant Keys

-   Store duplicate keys entries together in the hash table.

##### 3.1.1.2 Observation
To reduce the # of wasteful comparisons, it is important to avoid collisions of hashed keys.

This requires a hash table with ~2x the number of slots as the number of elements.

#### 3.1.2 Robin Hood Hashing
Variant of linear hashing that steals slots from "rich" keys and give them to "poor" keys.

-   Each key tracks the number of positions they are from where its optimal position in the table.
-   On insert, a key take the slot of another key if the first key is farther away from its optimal position than the second key.

#### 3.1.3 Cuckoo Hashing
Use multiple hash tables with different hash functions.

-   On insert, check every table and pick anyone that has a free slot.
-   If no table has a free slot, evict the element from one of them and then re-hash it find a new location.

Look-ups and deletions are always O(1) because only one location per hash table is checked.

Make sure that we don't get stuck in an infinite loop when moving keys.

If we find a cycle, then we can rebuild the entire hash tables with new hash functions.

-   With two hash functions, we (probably) won't need to rebuild the table until it is at about 50% full.
-   With three hash functions, we (probably) won't need to rebuild the table until it is at about 90% full.

### 3.2 Daynamic Hashing Schemes

The previous hash tables require knowing the number of elements you want to store ahead of time.

-   Otherwise you have rebuild the entire table if you need to grow/shrink.

Dynamic hash tables are able to grow/shrink on demand.

#### 3.2.1 Chained Hashing
Maintain a linked list of buckets for each slot in the hash table.

Resolve collisions by placing all elements with the same hash key into the same bucket.

-   To determine whether an element is present, hash to its bucket and scan for it.
-   Insertions and deletions are generalizations of lookups.

The hash table can grow infinitely because you just keep adding new buckets to the linked list.

You only need to take a latch on the bucket to store a new entry or extend the linked list.

#### 3.2.2 Extendible Hashing
Chained-hashing approach where we split buckets instead of letting the linked list grow forever.

This requires reshuffling entries on split, but the change is localized.

#### 3.2.3 Linear Hashing
Maintain a pointer that tracks the next bucket to split.

When any bucket overflow, split the bucket at the pointer location.

Overflow criterion is left up to the implementation.

-   Space Utilization
-   Average Length of Overflow Chains

Splitting buckets based on the split pointer will eventually get to all overflowed buckets.

-   When the pointer reaches the last slot, delete the first hash function and move back to beginning.

The pointer can also move backwards when buckets are empty.

## 4 Conclusion

Fast data structures that support O(1) look-ups that are used all throughout the DBMS internals.

-   Trade-off between speed and flexibility.

Hash tables are usually not what you want to for a table index.
