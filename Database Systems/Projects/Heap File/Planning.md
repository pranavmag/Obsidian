2026-09-10 21:48

Tags:

### Unordered Heap Files

Unordered heap files store records without any regard to order. allocated/deallocated as the file shrinks/grows. The records are appended to an available slot on a page.

To support record level operations we have to keep track of the pages in a file, free space on pages, and records on a page.

We can use a page directory 

A heap file consists of many slotted pages. Slotted pages are where the slot array maps "slots" to the tuples' starting position offsets. The header keeps track of the number of used slots and the offset of the starting location of the last slot used. The slot should store the offset and length to show where the record phyically lives.

The header and slot directory starts from the top of the page and records grow up from the bottom of the page, and free space is between them. So if our page is 4096 bytes then the slot directory starts near byte 0 and records start near byte 4096.

The heap file owns/manages many heap pages and the heap page interprets one 4096 byte page as header, slot directory, records. I think it's important to make that distinction now in the architecture.

The heap file can store these heap pages as a linked list or a page directory but the page directory is a better approach.

#### Heap Page

So we need to store 
