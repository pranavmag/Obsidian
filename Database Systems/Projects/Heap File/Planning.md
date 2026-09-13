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

move rest of slots below slot 1 by the length of slot 1.

slot 0 <--- slot 2 <--- slot 3 <--- slot 4 <--- slot 5

now later slot 1 can be reused but will be at a different offset in the file.

insert a new record, now it will take slot 1.

slot 0 <--- slot 2 <--- slot 3 <--- slot 4 <--- slot 5 <--- slot 1

```


