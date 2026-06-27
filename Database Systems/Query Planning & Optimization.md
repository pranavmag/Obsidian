We have a query where the user wants to know the names of all employees who work in the 'Toy' department.

```
SELECT DISTINCT ename 
FROM EMP E JOIN Dept D
ON E.did = D.did
WHERE D.dname = 'Toy'
```

We can make a relational algebra tree based off the query. So we know that two tables are being joined here which reflects the cross product. That would be our first thing that happens. Then we are selecting from these two tables with Emp.did = Dept.did. Then we select from the rows where the department name is 'Toy'. Finally we use a projection to extract just the employee name column.

**Product (Emp $\times$ Dept)

**Selection (**$\sigma_{Emp.did = Dept.did}$**)**

**Selection (**$\sigma_{dname = 'Toy'}$)

**Projection (**$\pi_{ename}$)

This is a literal translation of that SQL Query but it is a very poor representation to use. If Emp has 10,000 records and Dept has 500 records, then the cartesian product creates a temporary table with 5,000,000 rows and throws it away right after in the next step. 

This is likely where Query Optimization comes in. We'll take this baseline algebra tree and rearrange it into its most optimized state. For example, I think a good database engine would put the $\sigma_{dname = 'Toy'}$ at the bottom to filter out the Dept table to only a couple of rows ideally. Then the cross product would millions of rows less than it was with the unoptimized version.

Another fix may be to choose smarter physical operators like using a Page Nested Loop Join instead of the simple Cartesian Cross Product. The relational algebra just says what to do but the database engine has to choose how to physically execute it.

So if the catalog tells us that Emp takes up 1,000 pages on disk and Dept takes up 50 pages then we know that a simple tuple nested loop would read a single employee record, then scan the entire Dept table looking for matches, doing it 10,000 times. It would be 1000 pages + (10,000 records x 50 pages) =  501,000 Disk I/Os.

The database could instead use a Page Nested Loop Join which reads an entire page of Employee records into memory, and then scans the Dept table to find matches for everyone on that page at once. It would be 1000 pages + (1,000 records x 50 pages) =  51,000 Disk I/Os.

### Logical vs. Physical Plans

This from how I see it is basically the what and the how. The optimizer generates a mapping of a logical algebra expression to the optimal equivalent physical algebra expression. The physical operators define a specific execution strategy using an access path. The logical plan i would guess to always be abstract and the physical plan would take those abstract math concepts and turn them into specific, executable code algorithms.

It's important to note that it is not always a 1:1 mapping from logical to physical. As we saw, a single logical operator can be executed by several different physical algorithms depending on the situation.

The physical format still matters though because the physical operators depend on data format (sorting, compression). If your data is already sorted on the disk, the Optimizer will choose the lightning-fast Sort-Merge Join. if your data is highly compressed, the Optimizer has to factor in the CPU cost of uncompressing that data before it can read it.

### The Query Optimization Loop

The optimizer's job is to look at our simple Logical tree, generate dozens of possible Physical algorithm combinations, calculate the estimated cost of each, and execute the winning physical plan. It looks like this can get pretty complicated though.

It is NP-Hard meaning that the math problem gets so complex so fast that no computer can solve it perfectly in a reasonable amount of time. If we use Join Ordering as an example, if we want to join 2 tables there's only 2 ways to order them, but as you add more the logical arrangements increases fast. 3 tables -> 12 possible logical trees, 10 tables -> over 17 billion.

(n−1)! × Catalan(n−1)

Yeah this seems like a lot as you even add on more tables, it could take hours to just come up with a search plan. That's even before multiplying that number by the amount of physical algorithms you could use for each step.

To pick the cheapest plan it uses statistics stored in the database catalog to make incredibly educated guesses. 

It can use histograms and data distribution to track the shape of the data like if it knows that 90% of our employees are in the 'Sales' department vs. only 1% in the 'Toy' department.

It can also use cardinality estimation to predict how many rows our query will return with the filters. Like if we have `WHERE age > 100` and it knows that it will only return 5 people out of a million, it expects the next step in the tree to be very fast and cheap.

It can also use hardware math by assigning mathematical weights to operations. Like 1 disk read = 100 points of cost, 1 CPU calculation = 1 point of cost.

Practically: Choose from a subset. The optimizer uses heuristics (rules of thumbs) to aggressively throw out bad options without doing the math. 

**Predicate Pushdown:** It creates a hard rule: _Always_ do filtering (`WHERE`) as early as possible. It won't even bother evaluating a plan that filters _after_ a join.

**Restricting Tree Shapes:** Many optimizers refuse to look at complex "bushy" trees (joining A and B, then joining C and D, then joining the results). They only look at simpler "left-deep" trees, which drastically cuts down the search space.

**Timeouts:** Sometimes, the optimizer just sets a timer. If it finds a "good enough" plan in a few milliseconds, it stops searching the space entirely and just runs it!





