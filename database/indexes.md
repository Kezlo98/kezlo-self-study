# Index

![Indexes](/content/indexes.png)

## What are indexes?

- Is a special data structure that improves the speed of data retrieval operations on a database table.
- Without index, database has to scan through the entire of table sequentially to find the matching record => Time complexity: O(n)
- But n only increase time by time, and it normal huge, can be > 1M => performance issue

## Why an index can provide faster retrieval compared to a search without an index

- **Reduced Data Access**: Database can directly locate the relevant data pages or rows containing the desired values => Doesn't need to scan the entire table => significantly reduce data access and I/O operations
- **Smaller Search Space**: The index structure provides a narrower search space => quicker identification of the desired records
- **Optimized Disk Access**: Because an index typically resides in a separate data structure, it is designed to be more compact and fit in memory => requires fewer disk I/O operations
- **Search Algorithms**: Indexes employ efficient search algorithms like binary search,... which have time complexity lower than linear => Improve retrieve performance

## Why don’t we index every column to support fast read?

Because it will have the following bad effects:

- INSERT to the table will be slower since index must be updated
- The table's storage footprint will be larger as all these extra indexes need to be stored
- The query optimization process will be slower as there's more possible query plans to analyze (event if some not useful)

## Disadvantage

- Additional Writes operation
- Storage space to maintain the index data structure

## Index type

1. **Hash Indexes**

    - ![hash](/content/Hash.png)
    - Based on a Hash Table
        - **key**: **hash code** of the indexed columns
        - **value**: Pointer to the corresponding row (**page_id, slot_id**)
    - ***Advantages***
        - O(1) in both reads and writes data
        - 0(1) for equality query (=, !=, IN)
    - ***Disadvantages***
        - **Hash indexes store in memory**
            - Data not store close in disk => not fast while using disk => use ram to store
            - Hash indexes store in memory => key need to fit in RAM => expensive
            - RAM is not durable => need write-ahead log store in disk
            - If we're losing RAM => need to create Hash again => run all log => cost time
        - **No range queries**
            - Due to hash function => cannot compare => cannot use range queries
        - **No support partial key matching**:
            - If index(***Col1, Col2***) => not help if query only with ***Col1*** or ***Col2***
        - **Performance can bo unstable in case of hash collision**

2. **B-Tree**

    ![B-Tree](/content/B-TreeIndexes.png)

    - Based on a Binary Search Tree
        - Store in disk
        - Auto balanced
        -
    - B-Tree properties:
        - Self-balanced: every leaf node is at same depth
        - Each node has M-1 keys and M children
        - Each node (except root) is at least half-full: M/2 - 1 <= keys <= M - 1
        - Keys in each node are sorted
        - Leaf nodes are connected 2-ways
    - **General Structure**
      ![B-TreeNode](/content/B-TreeNode.png)
    - **B-Tree Search**
        - Start from root, search correct child and follow the pointer
        - Because of keys are sorted => can use Binary Search
        - Runtime: ~O(logN)
    - **B-Tree Insert**
        - Find the correct **Leaf node L** (Search)
        - Put the new key into L in sorted order
        - If L has enough space => done!
        - If not => split L into 2 **even** children L1 and L2, push up the middle key to L's parent. Repeat if L's parent if full
        - Runtime:
            - time to splitting key: O(M) (M: number children of current node)
            - insert time: ~ O(logN)
    - **B-Tree vs B+Tree**
      ![B-TreeVsB+Tree](/content/B-TreeVsB+Tree.png)
        - B-Tree different:
            - Inner Node can contain pointer to records
            - no duplicated key
            - leaf-node are not connected as D-Linked List
        - B+Tree over B-Tree:
            - Support full scan:
                - Leaf nodes of B+ are linked => full scan of all objects in a tree requires just one linear pass through all the lead nodes
                - Inner node of B+ does not have a pointer to data => more keys can be packed in a B+ => reducing the tree's height
                  ![B+TreeDataPointer](/content/B+TreeDataPointer.png)
    - ***Advantages***
        - Match leftmost prefix index:
            - Index (Col1, Col2, Col3) => B+Tree index can support query on (Col1), (Col1, Col2), (Col1, Col2, Col3)
        - Match part of the first column: Index on **Name** column => B+ index can support query all records having **Name** start with letter "Qu"
        - Match a range of values (>=, >, <=, <)
        - Store on disk => not easy to lose data
    - ***Disadvantages***
        - More complex than Hash indexes
        - Write speed is slower than LMS-tree indexes(SSTable)
        - Slower search than Hash index for equality query (=, !=, IN)

    - ***Why use B+ for index, not BST/AVL/Red-Black Tree ?***
        - B+tree are self-balance while BST is not => can lead to a very tall tree
        - B+ are much shorter than other balanced binary tree structures such as AVL => fewer disk access
        - B+ each node contains a large number of key sorted => much more efficient than AVL, BST or Hash table for range query

3. Best Practice
    - Only Index on **isolating** columns
        - should **not** be part of an expression or be inside a function in the query
        - Ex: ```SELECT id FROM student WHERE id + 1 =5``` => not using index => full scan
    - Index only prefix if column is too large:
        - Avoid index very long characters
        - Can index just first few characters
    - ```selectivity = number of distinct indexed values / number of rows in table``` => High selectivity is good
    - Should not index boolean column => ```selectivity = 2 / #rows``` => low
    - Use **Covering** index: indexes that contain all the data needed to satisfy a query

4. Cluster index vs Non-Cluster Index
    - Cluster index:
      - Is index which contain both key and record in same node. Should only contain 1 cluster index for each record.
      - Default primary key will be cluster index
    - Non-cluster index:
      - Is index which contain key and pointer to record