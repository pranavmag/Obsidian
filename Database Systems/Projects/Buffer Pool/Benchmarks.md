2026-10-27 21:16

Tags: 

The stats I want to benchmark on are fetch requests, cache hits, cache misses, evictions, disk reads, disk writes, and dirty evictions.

FetchPage() if a page is already resident should request and get a cache hit

FetchPage() if a page is not resident should request, get a cache miss, and then disk read

If we miss a required victim then we evict the page and if there is a dirty victim we have to evict and write to disk.

Whenever we flush a page we have to make sure that it's a write to disk and if we are reading a page from disk then it's a read from disk.

I first used a data set where the buffer pool size is 2, 4, 8, and 16 to check how each of these benchmarks act when the number of pages is 4 and the number of accesses is 1000. The data came back exactly as expected where if the buffer pool size is less than the number of pages then there will be 0 cache hits and 1000 cache misses so the buffer pool has to keep evicting pages to get the other pages in. 

But for the case of the buffer pool size being greater than or equal to the number of pages, the cache misses is equal to the number of pages because that is when these pages aren't resident in the buffer pool so those are mandatory disk reads. But after the 4 mandatory disk reads there are no other cache misses because all pages necessary are resident in the buffer pool.

The next part I would like to benchmark will be looking at what workloads does clock behave well or badly. We can probably do something like automated parameter sweeps and allow Python to benchmark across many combinations:

```
buffer sizes:     2, 4, 8, 16, 32, 64
working sets:     4, 8, 16, 32, 64, 128
accesses:         10000+
workloads:        sequential, random, hot/cold, looping
repetitions:      10
```

For example with hot/cold access and we have 100 pages and only 10 pages are hot pages that are frequently accessed and the other 90 are cold pages. Lets say that our buffer only holds 16 pages which means that we have significantly more pages than what can fit in the buffer, which in our sequential case would cause a lot of cache misses and evictions, but if we use hot pages we may have significantly less cache misses and evictions because the hot pages only take up 10/16 buffer pages and those are the ones that are most accessed.



