2026-08-24 17:40

Right now, I think I have a solid simple file manager here. I'm able to allow the file manager to write to and read from a page, but the main issue is that we're handling the state through CLI with user input. In actual systems, a real database engine running high-performance workloads cannot wait for a human to type in a block number. The system has to run completely autonomously. I want to build the `DiskMgr` class in a new file called `disk_mgr.hpp` and `disk_mgr.cpp` now to automate that state tracking.

The disk manager will be an abstraction layer on top of the file manager and keep track of the block count. So I probably need to implement an `Allocate()` function that creates new blocks and have another function `Append()` to write a page into it. The block id should be returned.



