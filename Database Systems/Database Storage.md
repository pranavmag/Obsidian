2026-07-11 19:11

Tags: 

The DBMS assumes that the primary storage location of the database is on non-volatile disk, meaning memory that retains data even when the power is removed. The DBMS's components manage the movement of data between non-volatile and volatile storage.

#### Storage Hierarchy

CPU Registers -> CPU Caches -> DRAM -> SSD -> HDD -> Network Storage
Faster, Smaller, Expensive ---------> Slower, Larger, Cheap

Volatile, Random Access, Byte-Addressable: CPU Registers, CPU Caches, DRAM
Non-Volatile, Sequential Access, Block-Addressable: SSD, HDD, Network Storage

With memories that are byte-addressable you can grab any amount of memory by offsets whereas with block-addressable you can only grab by pages. 

From now on lets address CPU for CPU Registers and CPU caches, Memory for DRAM, and Disk for SSD, HDD, and Network Storage

#### Access Times

1 ns - L1 Cache Ref
4 ns - L2 Cache Ref
100 ns - DRAM
16,000 ns - SSD
2,000,000 ns - HDD
~50,000,000 ns - Network Storage
1,000,000,000 ns - Tape Archives

### Sequential vs Random Access

Random access on non-volatile storage is almost always much slower than sequential access.

DBMS will want to maximize sequential access because algorithms will try to reduce number of writes to random pages so that data is stored in contiguous blocks. Allocating multiple pages at the same time is called an extent.

### Disk-Oriented DBMS

A database file is on disk and the file is broken up into multiple pages as a way to divide up the data. In memory we have a buffer pool, the memory managed by the DBMS that is used to bring pages from disk into memory. The execution engine is the query engine that executes SQL queries and knows how to read and write data from the buffer pool manager.

### Storage Engine

Local queries coming from clients or from other nodes are executed by the storage engine. It has a couple components.

Transaction manager - Schedules transactions and ensures they cannot leave the database in a logically inconsistent state

Lock manager - Locks on the database objects for the running transactions, ensuring that concurrent operations do not violate physical data integrity

Access methods (storage structures) - Manages access and organizing data on disk, access methods include heap files and storage structures such as B-Trees or LSM Trees

Buffer manager - Caches data pages in memory

Recovery manager - Maintains the operation log and restoring the system state in case of a failure

Transaction and lock managers are responsible for concurrency control, guaranteeing the logical and physical data integrity while ensuring that concurrent operations are executed as efficiently as possible.

### Memory- vs Disk-Based DBMS

In-memory DBMS (main memory DBMS) store data mainly in memory and use the disk for recovery and logging whereas disk-based DBMS store most of the data on disk and use memory for caching disk contents or as a temporary storage.

Memory-based DBMS exist for better performance and access granularity. Programing for main memory is also simpler than programming for disk because of the operating system abstracting memory management.

The limiting factors of In-memory databases are RAM volatility and costs. RAM contents are not persistent making them more susceptible to data loss. Ways to combat this usually require more hardware and expertise. Disks are easier to maintain and have significantly lower prices. 

In-memory database systems hold backups on disk. Before each transaction or write operation is completed, the results are written to a sequential log file. From my research, a sequential log file is an append only file where the records of database operations are written one right after the other in the order that they occur. It follows the literal naming of it.

A backup copy of the log contents are stored, taking snapshots every so often so that if a crash occurs the log contents don't need to be replayed from the beginning. Logs can contain many months of information so replaying it from the beginning would take too long. Log records are usually applied to backup in batches and the backup holds a database snapshot for a specific point in time. Any log contents up to that point can be discarded. This is known as checkpointing and reduces recovery times significantly.

### Column- vs Row-Oriented DBMS

Tables can be partitioned either horizontally (storing values belonging to the same row together) or vertically (storing values belonging to the same column together). 

Row-oriented DBMS store data in records/rows with every row having the same set of fields. They are efficient at storing transactional data like user entries, names, and phone numbers. A record can be uniquely identified by the key. Since row-oriented databases access data by row, storing entire rows together improves spatial locality.

Since data on disk is accessed through blocks, a single block will contain all the fields for one record, then go on. So from what I understand, if we have `sid, name, age, email` then all fields associated with the record for sid 1 will be on a block and then if more data can fit then all data for sid 2 will be stored on the same block, and so on.

Column-oriented DBMS store data vertically by column instead of storing it horizontally by row. Values for the same column are stored contiguously on disk instead of rows. Storing values of different columns in separate files or file segments allows efficient queries by column since they can be read in one pass.

Column-oriented databases are great for analytical workloads that compute aggregates, such as finding trends, computing average values, etc. To reconstruct data tuples, some metadata may be preserved to identify each column value to other fields, so each value will hold a key. Some column stores use implicit identifiers (virtual IDs) instead and use the position of the value to map it back to the related values. 

I do tend to see this trend of using virtual versions of things usually in systems performance, I've seen it predominantely in the form of virtual memory where it can make the program think it has unlimited memory to work with but the actual memory is mapped properly later. I assume it's the same with virtual IDs, reducing performance overhead.

### Wide Column Stores

Wide column stores are a bit different, colums are grouped into column families and inside each column family the data is stored row-wise which can be retrieved by a key or a sequence of keys. 

### Data Files and Index Files

A database system wants a reliable way to store data and allow quick access to it, so the files are stored using implementation-specific formats rather than flat files. This allows for storage efficiency, access efficiency, and update efficiency.

Each record in a table can be identified using a search key as we know for example `sid`. These records are located using an index which are auxiliary data structures that allow database systems to locate data records withotu scanning an entire table on every access. Indexes are built using a subset of fields identifying the record.

I had to look up what an auxiliary data structure was and it just means that it's a supporting data structures that performs tasks that help manage, search, or process the primary data more efficiently. It does not store the original data so if it is deleted the database won't be losing data, it exists purely for performance.

Database systems usually separate data files and index files. The data files store data records while the index files store record metadata that is needed to locate records in data files. Files are partitioned into pages, which typically have the size of a single or multiple disk blocks. Pages can be organized as sequences of records or as a slotted pages.

From my Toy DB File Manager project, I recall that disks are block-addressable and pages are byte-addressable so I used offsets to read or write specific data. I will have to see if there are other implementations here in this book.

### Data Files

Data files (primary files) can be implemented as index-organized tables (IOT), heap-organized tables (heap tables), or hash-organized tables (hashed files).

Records in heap files don't follow any order specifically but are usually stored in write order so no additional file reorganization or restructuring is needed when new pages are appended. Heap files do need an extra index structure that points to the locations of data records to make them searchable.

In hashed files, records are stored in buckets and the bucket that a record blongs to depends on its hash value of the key. Records in buckets can be stored in append order or sorted by key to improve lookup speed.

IOTs store data records in the index itself and since records are stored in key order, range scans in IOTS can be implemented by sequentially scanning its contents. This method saves at least one disk seek because after traversing the index and locating the searched key we don't have to address a separate file to find the associated data record because the data records are embedded directly within the index's structure. A disk seek is one of the slowest and most expensive operations so this is a benefit.

Although there are some other tradeoffs it seems which makes it so that not all database systems just use IOTs as their default. For example, I looked into it and found out that PostgreSQL uses heap tables as the native storage structure. I looked into potential reasons why and it may be because IOTs have overhead during inserts because of needing to find the correct physical position on the B-tree to maintain the sorted order whereas in heap files data is just appended to the first available free space. So it seems that IOTs may potentially need a lot of reorganizing and such to ensure that the sorted order is correct.

### Index Files

An index is a structure that organizes data records on disk in a way that is efficient for retrieval operations. Index files map keys to locations in data files where the records identified by these keys or primary keys are stored. An index on a primary file is known as a primary index and all other indexes are secondary and they can point directly to the data record or store its primary key. Primary index files hold a unique entry per search key whereas secondary indexes may hold several entries per search key.

If the order of data records follows search key order it is called a clustered index and are usually stored in the same file or in a clustered file where the key order is preserved. The opposite is a non-clustered index.












