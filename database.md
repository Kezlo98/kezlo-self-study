# Database

## Database Management System (DBMS)

**DBMS** is a software application that enables users to efficiently store, organize, manage, and retrieve data from database.

### File Storage

- Database store on disk as a bunch of **files**
- DBMS is the one that maintains the database files
- It organizes files as a collection of **pages**
- ![Database File](content/DatabaseFile.png)
  
### Pages

- Is a **fixed-size** block of data:
  - Contain record (row + column), meta-data, indexes, .....
  - Default Pagesize:
    - MySQL: 16KB
    - SQL server, PostgreSQL: 8KB
    - MongoDB: 32KB
- In a page, all records belong to the same table
- When BDMS reads data, it reads by page
- When database save data => save into pages
- MySQL: max size of row ~ 1/2 page's size = 8KB (InnoDB engine)
- If we have 1 row contain text = 7KB => Where to store ? **Store in different space and save url into db**

### Page Layout

- Use ***Slotted page layout*** to store:
  - At the beginning of the Page, right after the Header, it store an array that contain offsets to tuple(data) => called Slot Array
  - The tuples (data) will be at the end of **Page**
  - ![Page Layout](content/PageLayout.png)
  - When we need to insert a new tuple:
    - Read the starting location of the last slot used
    - Create a new slot in slot array
    - write the new tuple to the position

### Data page

- Include 2 part: ***header*** and ***data***
- Header will contain metadata:
  - Page size
  - Checksum
  - DBMS version
  - Number of empty slot
  - ....
  - ![Data Page](content/DataPage.png)

### Heap File Organization

- Is a common way DBMS organizes database files on disk
- Each file is an **unordered** collection of pages
- A special page called **Directory page**:
  - Directory page will be at the head of the file
  - Use to keep track of the location of data pages in the file
  - Use to keep tract the number of free slots per page
  - Mapping from **Page_id** to its position(offset) in the file => increase read speed
  - ![HeapDirectoryPage](content/HeapDirectoryPage.png)

### Buffer Pool (== Cached)

- A cache layer in memory (RAM), organized as an array of fixed-size pages
- Has Frame which **Frame's size == Page's size**
- Has Page table to map from **page_id** to **frame_id**
- DBMS use Buffer Pool to improve the performance
- When BDMS request to read a page:
  - Finds the page in **Buffer Pool**
  - If found => return
  - If not => **read from Disk**, copy to **Buffer Pool**, return
- ![Buffer Pool](content/BufferPool.png)
- **Replacement Policies**:
  - Pool's size is limited <= RAM's size (80% RAM)
  - DBMS needs to clear a frame to make room when full
  - Replacement policies is the way it decides which page to evict from the BP:
    - FIFO
    - LRU (Least Recent Use - Implement by using HashMap + LinkedList)
    - ...

## Index

![Indexes](content/indexes.png)

### What are indexes?

- Is a special data structure that improves the speed of data retrieval operations on a database table.
- Without index, database has to scan through the entire of table sequentially to find the matching record => Time complexity: O(n)
- But n only increase time by time, and it normal huge, can be > 1M => performance issue

### Why an index can provide faster retrieval compared to a search without an index

- **Reduced Data Access**: Database can directly locate the relevant data pages or rows containing the desired values => Doesn't need to scan the entire table => significantly reduce data access and I/O operations
- **Smaller Search Space**: The index structure provides a narrower search space => quicker identification of the desired records
- **Optimized Disk Access**: Because an index typically resides in a separate data structure, it is designed to be more compact and fit in memory => requires fewer disk I/O operations
- **Search Algorithms**: Indexes employ efficient search algorithms like binary search,... which have time complexity lower than linear => Improve retrieve performance

### Why don’t we index every column to support fast read?

Because it will have the following bad effects:

- INSERTs to the table will be slower since index must be updated
- The table's storage footprint will be larger as all these extra indexes need to be stored
- The query optimization process will be slower as there's more possible query plans to analyze (event if some not useful)

### Disadvantage

- Additional Writes operation
- Storage space to maintain the index data structure

### Index type

1. **Hash Indexes**

![hash](content/Hash.png)

- Based on a Hash Table
  - **key**: **hash code** of the indexed columns
  - **value**: Pointer to the corresponding row (**page_id, slot_id**)

- ***Advantages***
  - O(1) in both reads and writes data
- ***Disadvantages***
  - **Hash indexes store in memory**
    - Data not store close in disk => not fast while using disk => use ram to store
    - Hash indexes store in memory => key need to fit in RAM => expensive
    - RAM is not durable => need write-ahead log store in disk
    - If we losing RAM => need to create Hash again => run all log => cost time
  - **No range queries**
    - Due to hash function => cannot compare => cannot use range queries
  - **No support partial key matching**:
    - If index(***Col1, Col2***) => not help if query only with ***Col1*** or ***Col2***
  - **Performance can bo unstable in case of hash collision**

2. **B-Tree**

![B-Tree](content/B-Tree%20Indexes.png)

- Based on a Binary Search Tree
  - Store in disk
  - Auto balanced
  -  