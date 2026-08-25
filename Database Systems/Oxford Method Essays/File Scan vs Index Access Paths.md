2026-08-22 23:34

Tags: 

**Essay Question**: 

Consider this query:

```sql
SELECT * FROM Reserves WHERE bid = 100
```

`Reserves` has 100,000 tuples spread across 1,000 pages. There's an **unclustered hash index** on `bid`. Suppose `bid = 100` matches exactly 1,000 tuples — that's 1% of the table.

Two colleagues disagree:

- **Colleague A:** "We have an index that exactly matches this condition. Obviously we use it — that's the whole point of building an index."
- **Colleague B:** "Don't bother. Just scan the file top to bottom and check each tuple."

**Argue for one of them.**


### My Answer

Colleague B is right, it is much better to just scan the file top to bottom and check each tuple. This is because the index is unclustered and the range scan may have each tuple on each page. The formula is NTuples x 1/NKeys = 100,000 x 1/100 = 1000 tuples and the I/O cost will be 1000 tuples x 1 random read per tuple. So it would end up doing 1000 disk I/Os when it has 1000 pages anyways. The optimizer is smart enough to know to just do a full table scan. If this was a clustered index then it would only be 10 I/Os through the equation NPages x 1/NKeys = 1000 / 100 = 10 pages. So in this case the best choice is to just to do a full table scan. If the range for the unclustered index was much less then it would be reasonable to just use the index.


### Attack on my Answer

**The contradiction.** You computed the unclustered-index cost at ~1000 I/Os and the full-scan cost at ~1000 I/Os (since B = 1000 pages). Those are _the same number_. But you concluded the scan is "much better." Which is it — is colleague B's approach strictly cheaper, or are they roughly tied at this particular selectivity? Push further: does the index approach actually cost _exactly_ 1000 I/Os, or does it cost _more_ than that? Think about what it takes to even find those 1000 matching data entries in the first place — does probing a hash index to locate the right bucket cost you anything before you even start fetching data pages? If so, your own numbers should show the index losing, not tying.

**The sharper crossover question.** You gestured at "if the range was much less, index would be reasonable" — but I want the actual mechanism, not just the direction.


### My Response

Actually yeah, I forgot to consider the index probe which means that it would be 1,001 I/Os for using the index and 1,000 I/Os still for the full table scan. That doesn't seem like much but when a full table scan is done, it goes through one by one in a sequential continuous path whereas using the index would cause the database to jump randomly throughout the disk. Even if we assume that multiple tuples we need can be stored on the same page, the pessimistic upper bound possibility is still there. On paper we know that the fraction f of the table's tuples is 100,000 x f = 1,000 which makes it 1%. The two methods are equal but we know the performance overhead of using the index in this case so a full scan is better. I think that if f was 0.5% of the data then the index would win.