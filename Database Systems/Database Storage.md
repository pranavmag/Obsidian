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











