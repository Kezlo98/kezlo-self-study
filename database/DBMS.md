# Database Management System (DBMS)

**DBMS** is a software application that enables users to efficiently store, organize, manage, and retrieve data from database.

## File Storage

- Database store on disk as a bunch of **files**
- DBMS is the one that maintains the database files
- It organizes files as a collection of **pages**
- ![Database File](/content/DatabaseFile.png)

## Pages

- Is a **fixed-size** block of data:
    - Contain record (row + column), meta-data, indexes, .....
    - Default Pagesize:
        - MySQL: 16KB
        - SQL server, PostgreSQL: 8KB
        - MongoDB: 32KB
- In a page, all records belong to the same table
- When DBMS reads data, it reads by page
- When database save data => save into pages
- MySQL: max size of row ~ 1/2 page's size = 8KB (InnoDB engine)
- If we have 1 row contain text = 7KB => Where to store ? **Store in different space and save url into db**

## Page Layout

- Use ***Slotted page layout*** to store:
    - At the beginning of the Page, right after the Header, it store an array that contain offsets to tuple(data) => called Slot Array
    - The tuples (data) will be at the end of **Page**
    - ![Page Layout](/content/PageLayout.png)
    - When we need to insert a new tuple:
        - Read the starting location of the last slot used
        - Create a new slot in slot array
        - write the new tuple to the position

## Data page

- Include 2 part: ***header*** and ***data***
- Header will contain metadata:
    - Page size
    - Checksum
    - DBMS version
    - Number of empty slot
    - ....
    - ![Data Page](/content/DataPage.png)

## Heap File Organization

- Is a common way DBMS organizes database files on disk
- Each file is an **unordered** collection of pages
- A special page called **Directory page**:
    - Directory page will be at the head of the file
    - Use to keep track of the location of data pages in the file
    - Use to keep tract the number of free slots per page
    - Mapping from **Page_id** to its position(offset) in the file => increase read speed
    - ![HeapDirectoryPage](/content/HeapDirectoryPage.png)

## Buffer Pool (== Cached)

- A cache layer in memory (RAM), organized as an array of fixed-size pages
- Has Frame which **Frame's size == Page's size**
- Has Page table to map from **page_id** to **frame_id**
- DBMS use Buffer Pool to improve the performance
- When BDMS request to read a page:
    - Finds the page in **Buffer Pool**
    - If found => return
    - If not => **read from Disk**, copy to **Buffer Pool**, return
- ![Buffer Pool](/content/BufferPool.png)
- **Replacement Policies**:
    - Pool's size is limited <= RAM's size (80% RAM)
    - DBMS needs to clear a frame to make room when full
    - Replacement policies is the way it decides which page to evict from the BP:
        - FIFO
        - LRU (Least Recent Use - Implement by using HashMap + LinkedList)
        - ...
