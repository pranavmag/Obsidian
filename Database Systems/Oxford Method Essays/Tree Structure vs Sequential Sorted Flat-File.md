2026-08-19 22:43

Tags: 

**Essay Question:** You've just established that a fully sorted file is a bad idea because inserts require shifting large blocks of records. Suppose someone proposes this fix: "Keep the file sorted by age, but instead of storing it as one giant flat sequential file, organize it as a **tree structure** — like a B+ tree — where the leaf level holds the actual employee records in sorted order, and a hierarchy of index pages sits above it directing searches to the right leaf."

Someone objects: "That doesn't actually solve the insert problem. You still have to insert a new record into the correct sorted position. Restructuring is restructuring — whether you call it a flat file or a tree, you're still moving data around to keep things ordered."

**Argue against the objector.** Specifically: why would organizing the sorted data as a tree of pages (rather than one contiguous sequential file) make inserts cheaper, even though both structures maintain sort order?


### My Answer

A tree structure is way better than a flat-file when it comes to insertions. In a flat-file it has search through the entire file and rearrange almost every single record depending on the insert. With a tree structure like a B+ tree on the other hand, it only traverses down the tree, depending on if the value is greater than, less than, or in between internal node values, to the required area and writes if available or does a page split and rearranges a couple records. It does not shift the whole file and does a couple I/Os while maintaining sorted order.