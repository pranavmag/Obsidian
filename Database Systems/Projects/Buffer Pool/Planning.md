2026-09-02 20:44

Tags: 

The memory region is organized as an array of fixed-size pages. Each array entry is called a frame. When the DBMS requests a page, it puts an exact copy of the page into one of these frames.

Dirty pages are buffered and not written to disk immediately. They use the write-back cache which is where the buffer pool will set like a boolean flag probably like `is_dirty = true` and leave the page in RAM. So the disk is ignored until the buffer pool is full and needs to evict that page. This is pretty good because it can pile up all the updates to that record within that time and do just one I/O disk write rather than write to disk immediately each time.

There is also another thing I am familiar with, I am not exactly sure if I will implement this right now or within this project, but it is the Write-Ahead Log (WAL). It's basically a sequential log file that has inserts, updates, etc written on there. I think it's a good thing to have in the case of power losses, all the dirty pages you had in the buffer pool are gone and all the updates you had were never written to disk which is a problem.

The page table keeps track of pages that are currently in memory, usually a fixed-size hash table protected with latches to ensure thread-safe access. It can also have additional meta-data per page like a Dirty Flag, Pin/Reference Counter, and Access Tracking Information. The page table does not need to be stored on disk, it's an in-memory data structure that simply maps page ids to a copy of the page in buffer pool frames.

The buffer pool can get full so to make room for a new page, the DBMS has to decide which page to evict from the buffer pool. The DBMS uses a Least-Recently Used (LRU), meaning that a single timestamp of when each page was last accessed is stored. When the DBMS needs to evict a page, it selects the one with the oldest timestamp.

Another method is using the clock algorithm. Each page has a reference bit and when a page is accessed its bit is set to 1. When the buffer pool is full or a query asks for a page from the disk that isn't currently in memory, then it sweeps through the pages in a circular buffer with a "clock hand" that sweeps over pages in order. It checks the page's bit for whether or not it is set to 1. If it is set to 1 then it sets it to 0 and moves on, if it is at 0 already then it evicts the page. In strict LRU, just reading a page requires acquiring an Exclusive Latch to safely move linked-list pointers around. In CLOCK, reading a page just means flipping an integer from `0` to `1`. Because nothing is moving around in the data structure, dozens of threads can read pages and flip bits simultaneously with almost zero locking overhead. I might honestly try to implement this one just straight out, it seems much more elegant than LRU and solves its latch contention problem.


