2026-08-20 23:45

Tags: 

SQL queries are translated into an extended form of relational algebra, and query evaluation plans are represented as treesof relational operators, along with labels that identify the algorithm to use at each node.

### The System Catalog

We can store a table using any of the file structures and create one or more indexes (each stored as a file) on every table. A relational DBMS maintains information about every table and index that it contains. The descriptive information is itself stored in a collection of special tables called the catalog tables also known as the data dictionary, the system catalog, or just the catalog.

For what is stored in the system catalog, we have system-wide information like the size of the buffer pool and the page size and the following for individual tables, indexes, and views:

For each table: its table name, the file name (or some identifier), and the file structure (e.g., heap file) of the file in which it is stored, the attribute name and type of each of its attributes, the index name of each index on the table, and the integrity constraints on the table.

For each index: the index name and the structure (e.g., B+ tree) of the index, and the search key attributes.

For each view: its view name and definition

Statistics about tables and indexes are also stored in system catalogs and updated periodically, not on every modification of the tables). These are commonly stored:

Cardinality: The number of tuples NTuples(R) for each table R.
Size: The number of pages NPages(R) for each table R.
Index Cardinality: The number of distinct key values NKeys(I) for each index I.
Index Size: The number of pages INPages(I) for each index I. (For a B+ tree index I, we take INPages to be the number of leaf pages.)
Index Height: The number of nonleaf levels IHeight(I) for each tree index I.
Index Range: The minimum present key value ILow(I) and the maximum present key value IHigh(I) for each index I.

The catalogs also contain information about users, such as accounting information and authorization information, for example Nav User can modify the Reserves table but only read the Sailors table.

#### How Catalogs Are Stored

Catalogs are actually written as a collection of tables themselves. Let's take the example tables here:

Sailors(sid: integer, sname: string, rating: integer, age: real)
Reserves(sid: integer, bid: integer, day: dates, rname: string)

The catalog to store information about the attributes of these tables would look like:

|**att_name**|**rel_name**|**type**|**position**|
|---|---|---|---|
|sid|Sailors|integer|1|
|sname|Sailors|string|2|
|rating|Sailors|integer|3|
|age|Sailors|real|4|
|sid|Reserves|integer|1|
|bid|Reserves|integer|2|
|day|Reserves|dates|3|
|rname|Reserves|string|4|

### Operator Evaluation

There are many kinds of algorithms available for implementing each relational operator and for most operators no algorithm is universally superior. There are many factors involved in determining the best algorithm to use such as the size of the tables involved, existing indexes, etc.

An access path is a way of retrieving tuples from a table and consists of either (1) a file scan or (2) an index plus a matching selection condition. Consider a simple selection that is a conjunction of conditions of the form attr op value, where op is one of the comparison operators. Such selections are said to be in conjunctive normal form (CNF), and each condition is called a conjunct.

A conjunction is just the logical term for `AND` and a conjunct is simply one individual condition tied to another condition by an `AND`.

A hash index matches a CNF selection if there is a term of the form attribute = value in the selection for each attribute in the index's search key. For example we have a hash index H on the search key (rname, bid, sid), we can use the index to retrieve just the Sailors tuples that have all three conjucts that satisfy the condition. The condition in this case could be `rname = 'Nav' AND bid = 5 AND sid = 3`. But if the selection condition is `rname = 'Nav' AND bid = 5`, or some other condition that isn't bid then the index does not match.

If we build a hash index with a search key on (rname, sid, and bid), we are telling the database to create a hash function based off these three columns to generate a single bucket ID. If our `WHERE` clause only provides two of the three values then the database cannot run the required hash function.

In contrast, a tree index like a B+ tree can match a CNF selection if there is any comparison operator in the form attribute op value in the selection for each attribute in a prefix of the index's search key. The B+ tree can match both `rname = 'Nav' AND bid = 5 AND sid = 3` and `rname = 'Nav' AND bid = 5`. However, it would not match `bid = 5 AND sid = 3` because tuples are sorted primarily by rname.

When we create an index on (rname, bid, sid), the database sorts these columns in a hierachical chain: primary sort (rname), secondary sort (bid), tertiary sort (sid). This entire tree is sorted alphabetically by the sailor's first name which is why it must be included in the selection condition.

If we have an index on the search key (bid, sid) and the selection condition `rname = 'Nav' AND bid = 5 AND sid = 3`, we can use the index to retrieve tuples that satisfy `bid = 5 AND sid = 3`. These are the primary conjuncts and the number of tuples that satisfy these conjuncts (and whether or not the index is clustered) determines the amount of pages retrieved. The index being clustered or not matters because if it is unclustered, depending on the range it may have to scan a large percent of the table, which in that case a full table scan might just be performed instead to grab all pages. The additional condition on rname will then be applied to each retrieved tuple and will eliminate some of the retrieved tuples from the result.

#### Selectivity of Access Paths

The selectivity of an access path is the number of pages retrieved if we use this access path to retrieve al desired tuples, and the most selective access path is the one that retrieves the fewest pages. Each primary conjunct acts a filter on the table and the fraction of tuples in the table that satisfy a given conjunct is called the reduction factor. If there are several primary conjucts then the product of their reduction factors gives the fraction of tuples that satisfy all of them. For an exact match using a full search key, this is simply 1/NKeys. For composite keys, the optimizer assumes statistical independence and multiplies the individual reduction factors of each matched conjunct together which is NPages(Sailors) * 1/NKeys(H). If the system catalog lacks distinct-value statistics for a specific column, the engine prevents the calculation from failing by substituting a hardcoded default (traditionally 1/10).

For range queries (e.g., `day > value`), the optimizer calculates the reduction factor by assuming uniform data distribution and taking the ratio of the requested range over the total index range: $\frac{High(T) - value}{High(T) - Low(T)}$. Once the final reduction factor is determined, the physical I/O cost hinges entirely on disk layout. If the index is clustered, the factor is simply multiplied by the total number of pages ($NPages$); if unclustered, the optimizer pessimistically assumes it must execute one random disk page read for every single tuple retrieved and probably just opts to do a full table scan.

### Algorithms for Relational Operations

As we know, for a selection if the index is clustered the amount of I/Os are significantly less than the I/Os for a unclustered index. It is probably cheaper to scan the entire table instead of using the unclustered index if over 5% of the tuples are to be retrieved.

For a projection, the index being unclustered does not matter if the columns we are retrieving is what the index is on. We can just do an index-only scan if that's the case. We just mostly have to be concerned about whether or not duplicates are returned. If the query has the `DISTINCT` keyword then we use partitioning. Let's say we want to get (sid, bid) by projecting from `Reserves`. We can partition by scanning Reserves to obtain (sid, bid) pairs and sorting these pairs using (sid, bid) as the sort key. We can then scan the sorted pairs and easily discard duplicates, which are now adjacent. This usually takes about 2-3 passes, the optimized way would be one pass to read the table and the (sid, bid) pairs are written out so all other irrelevant columns are disregarded and then the second pass to read the table again and remove the duplicates.

For joins there are a couple of approaches. One way is uisng index nested loop joins (INLJs). This is an incremental, row by row algorithm that pairs a sequential scan of the outer table with index probes on an inner table. For every tuple retrieved from the outer table, the execution engine searches the inner table's index to locate matching records, followed by a direct random I/O fetch of the raw data block to retrieve any unindexed columns. While INLJ can suffer from massive random disk I/O bottlenecks if forced to evaluate a large, unfiltered dataset, its incremental nature makes it the absolute optimal choice when a highly restrictive `WHERE` clause is present. By filtering the outer table down to a tiny subset of rows, the engine only executes a handful of index probes, finishing the query almost instantly without needing to process the bulk of the inner table.

Sort-Merge Join (SMJ) is a bulk processing algorithm designed to efficiently join massive datasets by replacing expensive random I/O with much cheaper sequential disk reads. It operates by first applying an External Merge Sort to both tables—reading, chunking, and writing them back to disk to order them by the join key—before streaming the two sorted tables side-by-side to merge matching tuples. Because SMJ is an "all-or-nothing" operation, it carries a heavy upfront disk I/O cost to sort the data regardless of how many rows the final query actually needs. Consequently, query optimizers favor SMJ exclusively when joining large, unfiltered tables, as paying the fixed sequential sorting cost is significantly cheaper than forcing the hard drive to execute hundreds of thousands of random index probes.

Another method is a Hash Join which involves going through all the values in one table and applying a hash function to each value to create a hash table, then going through the second table and probing the hash for each value.







