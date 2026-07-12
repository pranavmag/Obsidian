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





