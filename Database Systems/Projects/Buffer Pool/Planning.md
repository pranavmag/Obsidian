2026-09-02 20:44

Tags: 

The memory region is organized as an array of fixed-size pages. Each array entry is called a frame. When the DBMS requests a page, it puts an exact copy of the page into one of these frames.

Dirty pages are buffered and not written to disk immediately. They use the write-back cache which is where the buffer pool will set like a boolean flag probably like `is_dirty = true` and leave the page in RAM. So the disk is ignored until the buffer pool is full and needs to evict that page. This is pretty good because it can pile up all the updates to that record within that time and do just one I/O disk write rather than write to disk immediately each time.

There is also another thing I am familiar with, I am not exactly sure if I will implement this right now or within this project, but it is the Write-Ahead Log (WAL). It's basically a sequential log file that has inserts, updates, etc written on there. I think it's a good thing to have in the case of power losses, all the dirty pages you had in the buffer pool are gone and all the updates you had were never written to disk which is a problem.

The page table keeps track of pages that are currently in memory, usually a fixed-size hash table protected with latches to ensure thread-safe access. It can also have additional meta-data per page like a Dirty Flag, Pin/Reference Counter, and Access Tracking Information. The page table does not need to be stored on disk, it's an in-memory data structure that simply maps page ids to a copy of the page in buffer pool frames.

The buffer pool can get full so to make room for a new page, the DBMS has to decide which page to evict from the buffer pool. The DBMS uses a Least-Recently Used (LRU), meaning that a single timestamp of when each page was last accessed is stored. When the DBMS needs to evict a page, it selects the one with the oldest timestamp.

Another method is using the clock algorithm. Each page has a reference bit and when a page is accessed its bit is set to 1. When the buffer pool is full or a query asks for a page from the disk that isn't currently in memory, then it sweeps through the pages in a circular buffer with a "clock hand" that sweeps over pages in order. It checks the page's bit for whether or not it is set to 1. If it is set to 1 then it sets it to 0 and moves on, if it is at 0 already then it evicts the page. In strict LRU, just reading a page requires acquiring an Exclusive Latch to safely move linked-list pointers around. In CLOCK, reading a page just means flipping an integer from `0` to `1`. Because nothing is moving around in the data structure, dozens of threads can read pages and flip bits simultaneously with almost zero locking overhead. I might honestly try to implement this one just straight out, it seems much more elegant than LRU and solves its latch contention problem.

### Design

A Page should be an array of bytes for its data and should have a page_id.

Page
├── page_id
└── array of bytes (data)

Page
├── access data
└── access page_id


The Page Table shouldnt actually do much other than keep track of the mappings we have between pages and frames. It shouldn't add pages to a frame or evict them or anything like that. I think the best way to do it is using a hashmap (pageid -> frameid).

PageTable
└── hashmap (page_id -> frame_id)

PageTable
├── add mapping
├── look up mapping
└── remove mapping


The Buffer Pool has many frames and each frame holds a page. A frame of course needs its id and also the actual Page to put it in, as well as some additional meta-data like the pin_count which is a reference counter that tracks the number of threads or requestors currently accessing a specific page stored in a frame. It also should store a boolean flag that indicates whether a page has been modified while in the frame.

Frame
├── frame_id
├── Page
├── pin_count
└── is_dirty

Frame
├── access page
├── access frame_id
├── access/modify pin_count
└── access/modify dirty state


The Clock Replacer should know two things: the reference bit per frame and a clock-hand position. It should iterate through the frames and have the clock hand going around, and once the clock hand reaches the starting position without finding a page to evict we should return something that represents none. The Clock Replacer should check if a frame is currently pinned and whether or not the reference bit is 1. If it is 1, set the reference bit to 0 and if it is 0 already then evict the page.

ClockReplacer
├── reference bits per frame (std::vector<uint8_t>)
└── clock hand

ClockReplacer
└── find victim (iterate through frames)


The Buffer Pool should know the page, the frame to put it in, whether or not the Buffer Pool is full, be able to update the Page Table, and ask the Clock for a page to evict. It should be able to handle pinning of pages and flushing of pages.

BufferPoolManager
├── array of frames
├── page_table
├── disk_manager
└── clock_replacer

BufferPoolManager
├── cache a frame
├── fetch page
├── unpin page
├── flush page
└── flush all pages

### Concurrency

Two threads should be allowed to call FetchPage(3) concurrently. The BPM must synchronize them so page 3 only gets loaded once.

One thread should be allowed to call FetchPage() while another thread calls UnpinPage() but the pin count does need synchronization. A lost update hazard could occur if it doesn't have synchronization.

```
Thread A: read 1
Thread B: read 1

Thread A: calculate 2 (fetch 1 + 1 = 2)
Thread B: calculate 0 (unpin 1 - 1 = 0)

Thread A: write 2
Thread B: write 0 // Thread A's update is lost because B wrote last
```

We can use either atomic types or a mutex to resolve it.

The same issue occurs with dirty state for pages so it also needs synchronization. So both pin count and dirty flag are shared state.

Eviction of a page can't happen while another thread is using/modifying the frame, this is intuitive because if the pin count is greater than 0 it can't be evicted anyways. So a dirty unpinned page can be evicted but must be flushed first of course.

Shared state requiring protection: page table, frame metadata, pin count, dirty flag, clock state, and page contents

Operations requiring coordinated/atomic state transitions: FetchPage, UnpinPage, victim selection + eviction, FlushPage






