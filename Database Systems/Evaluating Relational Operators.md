2026-09-24 22:40

Tags: 

Let's take an example of a selection:

```
SELECT * FROM Reserves R
WHERE R.rname = 'Nav'
```

If we have no index on R.rname, and the data is not sorted on R.rname then the most selective access path is a file scan. For each tuple, the condition R.rname = 'Nav' is evaluated and if the condition is satisfied then that tuple is added to the result.

If we have no index on R.rname but the data is sorted on R.rname then we can do a binary search to locate the first tuple that satisfies the selection condition. To get any further tuples that may satisfy the selection condition we start at the location of the first matched tuple and scan R until the selection condition is no longer satisfied. The access method here is a sorted-file scan with selection condition R.rname = 'Nav'.

If a clustered B+ tree index is available on R.rname then the index can be used for the selection condition R.rname = 'Nav' but since it is an equality a hash index would be preferred. B+ tree indexes are the best use for when the selection condition contains an `op` that is not equality although it is still a good option even in the case of equality. If the B+ tree index is not clustered then the cost of using the index depends on how many tuples satisfy the selection.