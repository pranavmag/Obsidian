Relational Algebra is used with relational database models. Those database models have procedural queries which is where the query specifies the way to find  the desired result based on sets/bags.

This is the opposite of Non-Procedural (Declarative) queries that specify only what data is wanted and not how to find it. Those types of queries are based off Relational Calculus.

### Operations

Relational Algebra has fundamental operations to retrieve and manipulate tuples in a relation. This is based on set algebra that has unordered lists with no duplicates.

They use tuples because tuples can represent a set of attribute values in the relation. For example like Artist(name, year, country) which the database will represent with its respective rows and columns.

Each operator takes one or more relations as its inputs and outputs a new relation. We can chain operators together to make more complex operations.

The operators are:
- σ Select
    
- π Projection
    
- ∪ Union
    
- ∩ Intersection
    
- − Difference
    
- × Product
    
- ⋈ Join

### Select (σ)

Choose a subset of the tuples from a relation that satisfies a selection predicate. The predicate acts as a filter to retain only tuples that fulfill its qualifying requirement.

It can combine multiple predicates using conjuctions/disjunctions.
Pretty much row filtering

Syntax: $\sigma_{predicate}(R)$

R(a_id, b_id)

| **a_id** | **b_id** |
| -------- | -------- |
| a1       | 101      |
| a2       | 102      |
| a2       | 103      |
| a3       | 104      |

$\sigma_{a\_id='a2'}(R)$

| **a_id** | **b_id** |
| -------- | -------- |
| a2       | 102      |
| a2       | 103      |

SQL Query:
SELECT *
FROM R
WHERE a_id = 'a2';

### Projection (π)

Generates a relation with tuples that contains only the specified attributes like rearranging attributes' ordering, removing unwanted attributes, and manipulating values to create derived attributes.

Syntax: $\pi_{A1, A2,...,An}(R)$

R(a_id, b_id)

| **a_id** | **b_id** |
| -------- | -------- |
| a1       | 101      |
| a2       | 102      |
| a2       | 103      |
| a3       | 104      |


$\Pi_{b\_id-100,a\_id}(\sigma_{a\_id='a2'}(R))$

|**b_id-100**|**a_id**|
|---|---|
|2|a2|
|3|a2|

SQL Query: SELECT b_id - 100, a_id
FROM R
WHERE a_id = 'a2';

### UNION (∪)

Generate a relation that contains all tuples that appear in either only one or both input relations.

Syntax: $(R \cup S)$

R(a_id, b_id)

| **a_id** | **b_id** |
| -------- | -------- |
| a1       | 101      |
| a2       | 102      |
| a3       | 103      |

S(a_id, b_id)

| **a_id** | **b_id** |
| -------- | -------- |
| a3       | 103      |
| a4       | 104      |
| a5       | 105      |

$(R \cup S)$

| **a_id** | **b_id** |
| -------- | -------- |
| a1       | 101      |
| a2       | 102      |
| a3       | 103      |
| a4       | 104      |
| a5       | 105      |

SQL Query: (SELECT * FROM R)
UNION
(SELECT * FROM S);

### Intersection (∩)

Generate a relation that contains only the tuples that appear in either both input relations.

Syntax: (R ∩ S)

R(a_id, b_id)

| **a_id** | **b_id** |
| -------- | -------- |
| a1       | 101      |
| a2       | 102      |
| a3       | 103      |

S(a_id, b_id)

| **a_id** | **b_id** |
| -------- | -------- |
| a3       | 103      |
| a4       | 104      |
| a5       | 105      |

(R ∩ S)

| **a_id** | **b_id** |
| -------- | -------- |
| a3       | 103      |

SQL Query: (SELECT * FROM R)
INTERSECT
(SELECT * FROM S);



