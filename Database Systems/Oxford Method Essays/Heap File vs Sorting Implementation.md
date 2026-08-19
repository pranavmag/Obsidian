2026-08-18 17:06

Tags: 

**Essay Question:** You're designing storage for an `Employees` table. The two operations that matter most are:

1. Inserting new employee records (happens constantly, e.g., new hires)
2. Retrieving all employees in a given age range (a common report query)

You have exactly two options, and must pick one for the _entire_ file

- **(A)** Keep the file physically sorted by `age` at all times.
- **(B)** Keep the file as an unordered heap, and build a separate index on `age`.


#### My Answer

A heap file with a separate index on age is the best structure by far. A heap file is great for inserts by nature because it just chooses the next available slot to insert data whereas physically sorting the file by age would mean that the structure of the file potentially has to be reorganized on each insert. For example if we get a new employee who is 23 years old, all record of employees older than him have to be moved down by one which is a hefty operation. A heap file would not care about this whatsoever and just append his record to the next available space. It can be argued that physically sorting by age is much better than a heap file for retrieving employees in given age ranges but to address that drawback we have a separate index on age. Heap files are better at retrieving one record or all of the records but are costly for retrieving multiple records within a range, so the index can handle this by sorting the records by age. This index would be auxiliary too meaning that if it is deleted, no data loss occurs. We'll likely have a data entry being a (k, rid-list) pair that can return a list of data records from our rids that have the search key k. This is efficient for the kind of retrievals we need to do.


#### Attack on my Answer

You said the index "handles" the range-retrieval problem by sorting records by age, and that this is "efficient for the kind of retrievals we need to do." Walk me through _why_ that's efficient, mechanically.

Your index sits on top of a heap file. The heap file's physical layout has nothing to do with age — records with similar ages could be scattered across completely different pages, in no particular order. Now say your range query is "all employees aged 30–40," and that's, say, 15% of your workforce.

Your index will happily hand you a sorted list of matching data entries — key value plus rid. But each of those rids can point to a _different, essentially random page_ in the heap file. So to actually fetch the employee records themselves, how many disk I/Os are you doing? Is it one I/O per matching page, or something closer to one I/O per matching _record_?


#### My Response

You make a good point on that, option B is certainly flawed for these ranged query readings compared to Option A even with the auxiliary index. It does do one I/O per matching record unfortunately but compared to the catastrophic write operation for option A, option B is still far better. Option B can actually be slightly optimized by sorting the rids by page id and then getting the pages in sequential order. But, option A could have a performance overhead so bad that it is unviable for constant inserts. If we think about the scale of company sizes that have multiple divisions and constant hires, writing a new employee on a multiple gigabyte file with tens of thousands (or more) employees would be a nightmare for the system.


### Final Remarks

Ideally there is a third better option that the textbook will probably address soon.

Even the rid sorting by page doesn't fully solve option B's bottleneck because there will be many pages scattering the employees that we need from our query. All it should guarantee is that we don't fetch the page twice. It would be negligible if we have to end up scanning 95% of the heap pages anyways, at that point it would be much better to just do a full table scan from the start. I do think that in upcoming chapters I will see better options or safeguards around these. Both options were absolutely flawed, I just saw more viability in option B if I had to choose between the two.
