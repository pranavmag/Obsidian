2026-08-30 22:44

Tags: 

**Essay Question**: 

You're joining `Reserves` (100,000 tuples, 1,000 pages) with `Sailors` (500 pages), on `Reserves.sid = Sailors.sid`. `Sailors.sid` is a key, and there's a hash index on it.

You calculate two options:

- **Index nested loops join** (scan `Reserves`, probe the `Sailors` index for each tuple): costs roughly 100,000 × (1.2 + 1) = 220,000, plus 1,000 to scan Reserves = 221,000
- **Sort-merge join** (sort both tables on `sid`, then merge): costs a few thousand I/Os total — dramatically cheaper.

A colleague says: "This settles it. Sort-merge join is strictly better here — it's an order of magnitude cheaper. Index nested loops should basically never be used when an index isn't already... wait, we _do_ have an index. So why would anyone ever choose index nested loops when the numbers this clearly favor sort-merge?"

**Argue against your colleague.**


### My Answer

Index nested loops join are great for when at least one table is smaller or when we want just a subset of one table (using a WHERE clause) to join with the other table. Lets say in the example we only want 2000 tuples from Reserves to be joined with Sailors, using our index will make this efficient, the formula being B(T1) + T1(h+C) where B(T1) is the amount of pages and T1 is the amount of tuples and it is being multiplied by the average I/Os for a hash index and the probing. So we have 20 + 2000(1.2 + 1) which gives us about 4420 I/Os plus the scan of the Reserves table. With the sort-merge join we still need to sort the entire table first and then do comparisons for the small subset which would be more I/Os than the index nested loop join in this case.


### Attack on my Answer

**First**, you leaned on "let's say we only want 2000 tuples from Reserves" and priced fetching those at B(T1) = 20 pages. That 20-page cost is only cheap if those 2,000 qualifying tuples are retrievable via something efficient — like a clustered index on whatever WHERE-clause condition selects them. What happens to your whole argument if there's _no_ index at all on the condition that picks out those 2,000 tuples — if you'd have to scan the full 1,000-page Reserves table just to find them? Does index nested loops join still win in that case, or does your advantage quietly evaporate?

**Second**, and this is the sharper structural point I want you to name explicitly: you said sort-merge "still needs to sort the entire table first." Why is that non-negotiable for sort-merge, no matter how small the number of tuples you actually want in the final answer? Put differently — is there any way to ask a sort-merge join to only sort "the 2% of the table I actually care about," or does the _sorting step itself_ have no concept of selectivity at all? Compare that to index nested loops: what is it about that algorithm's structure — something to do with how it processes the outer table one tuple at a time — that lets it naturally skip work for tuples it never looks at? There's a specific one-word property the chapter uses for this. See if you can name it or describe it precisely before you read, since it's the real answer to "why would anyone use index nested loops when the raw numbers favor sort-merge."


### My Response

Even if we don't have a clustered index and can't fetch the 20 pages exactly, we'll have our full outer scan for 1000 I/Os and only trigger the index probe for the 2000 tuples which is still 4400 I/Os giving us 5400 I/Os in total. INLJ is incremental meaning that it evaluates one row at a time, so it drops the workload to match the selectivity of the query which is why it is so great for the restrictive WHERE clauses. Sort-merge join's sorting algorithm which is an external merge sort does not care about how many tuples you want to retrieve, it will always sort both tables fully. There is no way to get sort-merge join to only sort the 2000 tuples we want.


