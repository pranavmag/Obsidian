2026-08-30 22:37

https://github.com/pranavmag/sort-merge-join-cpp

This is an implementation of a join algorithm used in databases called sort merge join. This algorithm is a database operation used to evaluate equijoins (e.g., SELECT * FROM t1 JOIN t2 ON t1.id = t2.id). It is great for for handling 1-to-1, 1-to-many, and many-to-many relationships without dropping duplicate keys.

The program takes two tables (vectors), sorts them and then returns the cartesian product of all tuples where t1.id == t2.id.

The time complexity of this algorithm for sorting is O(NlogN + MlogM + K) where N and M are the sizes of t1 and t2 and K is the size of the final joined output. The scanning is O(N + M).

There are two implementations, one in C++ and one in Rust.

