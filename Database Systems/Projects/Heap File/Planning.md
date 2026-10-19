2026-09-10 21:48

Tags:

### Unordered Heap Files

Unordered heap files store records without any regard to order. allocated/deallocated as the file shrinks/grows. The records are appended to an available slot on a page.

To support record level operations we have to keep track of the pages in a file, free space on pages, and records on a page.

We can use a page directory 

A heap file consists of many slotted pages. Slotted pages are where the slot array maps "slots" to the tuples' starting position offsets. The header keeps track of the number of used slots and the offset of the starting location of the last slot used. The slot should store the offset and length to show where the record phyically lives.

![[Slotted Page.png]]

The header and slot directory starts from the top of the page and records grow up from the bottom of the page, and free space is between them. So if our page is 4096 bytes then the slot directory starts near byte 0 and records start near byte 4096.

The heap file owns/manages many heap pages and the heap page interprets one 4096 byte page as header, slot directory, records. I think it's important to make that distinction now in the architecture.

The heap file can store these heap pages as a linked list or a page directory but the page directory is a better approach.

#### Heap Page

So we need to make HeapPage kind of like a wrapper over a Page. We can set the header and slot entry here and move the bytes from page over here to work over, using a memcpy. If we need to write the bytes back into the header or slot entry it's the same process. I think this separates concerns pretty well and keeps our `Page.h` as simple as we need it to be.

When initializing our Heap Page Header we have our 

Unlike our header that starts at byte 0, our slot 0 starts at the sizeof(header), slot 1 starts at sizeof(header) + sizeof(slot entry), etc. So our slots follow `sizeof(header) + N * sizeof(entry)`.

For records, we should have operations for inserting, deleting, getting, and updating them. For inserting we should be getting the record size, the required space for this record, checking if there is enough free space, the slot id, the new record offset, and then move the bytes from the record into the page at that offset, and of course writing back to the slot.

```
┌──────────────────────────────────────────────────────────┐
│  PageHeader                                              │
├──────────────────────────────────────────────────────────┤
│  Slot Directory  (grows → right)                         │
├──────────────────────────────────────────────────────────┤
│                                                          │
│              FREE SPACE (the "hole")                     │
│                                                          │
├──────────────────────────────────────────────────────────┤
│  Records  (grow ← left, from the back)                   │
└──────────────────────────────────────────────────────────┘
         ↑                                              ↑
   free_space_begin                           free_space_end
   (moves UP)                                (moves DOWN)   
   
So after each insert of a record we should move free_space_end down by the slot offset and move free_space_begin upwards by the size of a slot directory.
```

When deleting a record I want to use compaction to move all the slots in lower address spaces upward. For example if we have

```
higher address                             lower address
slot 0 <--- slot 1 <--- slot 2 <--- slot 3 <--- slot 4 <--- slot 5

record in slot 1 deleted

move rest of records below slot 1 by the length of slot 1.

slot 0 <--- slot 2 <--- slot 3 <--- slot 4 <--- slot 5

now later slot 1 can be reused but will be at a different offset in the file.

insert a new record, now it will take slot 1.

slot 0 <--- slot 2 <--- slot 3 <--- slot 4 <--- slot 5 <--- slot 1

```

### Heap File

A heap file should be able to store many heaps pages and know which pages belong to the file. It should be able to find/allocate pages for insertion and turn (page_id, slot_id) into a stable RID. It can fetch the right page for get/delete/update.

For a nice v1 I'll probably just get the basic CRUD operations going and not worry about HeapFile page ownership and persistence for `page_ids_` and free-space tracking until after the baseline v1 tests are established and passed. 

For inserting a record into a heap file we take the record and for each heap page we fetch the page and check whether or not the current record can fit. If it can fit we should insert it and get the slot_id and unpin the page as dirty and return the RID. If it can't fit then we should unpin the page as not dirty and have the bpm create a new page and initialize a heap page, insert our record into there and return RID. The appropriate latches should be used for each operation.

For getting a record we just take the RID and use the page_id and slot_id and fetch the appropriate page using that page id. We get the record using the slot_id and copy the bytes accordingly. The page can be unpinned as not dirty since no data was changed.

For deleting and updating a record it is essentially the same process except we unpin the page as dirty if the operation succeeded. Of course for updating a record we need the updated record and the RID passed in.

Also a big problem we have right now is that AddPage() in the buffer pool manager doesn't validate disk page ids before touching frame state so that should be changed. Also NewPage() needs to acquire a usable frame before allocating on disk. The buffer pool manager should be refactored for these changes.

Also the persistence of page_ids_ is the next step. For v1 I started off by using `std::vector<int> page_ids_;` for the page ids but if the process exits then the pages still exist but the page_ids_ are gone. We need to make this metadata persistent so that the heap file knows which disk pages belong to it even after restarting.

There's a couple ways to do this, I'm familiar with a linked list of these heap pages and a page directory with free-space metadata. I think I'll go with the page directory because it allows for faster inserts with variable-length records and also has a smaller metadata footprint. It only holds one entry per page (pointer + free space count) instead of two neighbor pointers embedded in every data page.

Design choices:

- Each directory entry represents one heap data page.
- `free_space` is cached metadata used to avoid fetching/scanning every heap page during insertion.
- `HeapPage::GetFreeSpace()` is the main value; the directory only caches that value.
- Directory metadata lives in normal database pages and therefore goes through the `BufferPoolManager`.
- `HeapFile` will keep only the ID of its first directory page in memory, rather than all data-page IDs.
- `next_dir_page` is included now so directory pages can be chained later, even if the first implementation initially uses only one directory page.
- A separate `HeapFileHeader` page is deferred for now. The first directory page acts as the persistent root/identity of the heap file.
- If a separate heap-file header is introduced later, it can store metadata such as `first_dir_page` and a `uint32_t page_count`.

The new path for `InsertRecord` should be fetching the first directory page, scanning the directory entries for enough free space, fetching the selected heap page, inserting the record, updating the directory entry's free space, and then marking the data page and directory page as dirty.

If no page has enough room then allocate a new heap page, initialize and insert the record, add the page directory entry, and store the remaining free space.

`HeapFile` decides which page to use; `HeapPage` decides how records are laid out inside that page.





