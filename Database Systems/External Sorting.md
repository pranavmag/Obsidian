2026-09-07 22:42

Tags: 

### Two-Way Merge Sort

In the first pass, the pages in the file are read in one at a time. After a page is read in, the records on it are sorted and the sorted page (a sorted run one page long) is written out. Quicksort or any other in-memory sorting technique can be used to sort the records on a page. In subsequent passes, pairs of runs from the output of the previous pass are read in and merged to produce runs that are twice as long. This algorithm is shown in Figure 13.1.

If the number of pages in the input file is 2k , for some k, then:

Pass 0 produces 2^k sorted runs of one page each,
Pass 1 produces 2^k-1 sortecl runs of two pages each,
Pass 2 produces 2 k - 2 sortecl runs of four pages each,
and so on, until
Pass k produces one sorted run of 2k: pages.

I have an example that makes this easier to understand. Let's say we have an unsorted file on the hard drive that is 4 pages long and that each page can hold 2 integer values. We have 3 buffer pages in RAM to work with.

- Disk Page 1: `[38, 12]`
    
- Disk Page 2: `[55, 9]`
    
- Disk Page 3: `[82, 41]`
    
- Disk Page 4: `[3, 19]`

Pass 0: The Initial Sort. We only use 1 RAM page.

- Read Page 1 into RAM. Sort it to `[12, 38]`. Write it to disk as **Run 1**.
    
- Read Page 2 into RAM. Sort it to `[9, 55]`. Write it to disk as **Run 2**.
    
- We repeat this for all pages.

Pass 1: The First Merge. We use all 3 RAM pages: Input A, Input B, and Output.

- Read Run 1 `[12, 38]` into Input A. Read Run 2 `[9, 55]` into Input B.
    
- The CPU compares the first numbers: 9 is smaller than 12. Move 9 to Output.
    
- Next: 12 is smaller than 55. Move 12 to Output.
    
- **The Hardware Bottleneck:** The Output page is now full `[9, 12]`. It flushes to the disk and clears itself so we can keep going.
    
- Continue comparing: 38 goes to Output, then 55. Output fills again `[38, 55]`, flushes to disk, and clears. We repeat this process for Run 3 and Run 4. The disk now has two sorted, 2-page runs: (`[9, 12]` followed by `[38, 55]`) and (`[3, 19]` followed by `[41, 82]`).

The passes continue on until all pages are sorted. The total amount of data the disk processes in every single pass is always exactly the same (the entire file), but the length of the individual sorted runs doubles with every pass.

Although this algorithm does not utilize buffer space effectively which external merge sort handles better.

### External Merge Sort










