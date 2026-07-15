2026-07-13 19:17

Tags: 

A relational database is a collection of relations, each relation being a table with rows and columns. This allows for a simple data representation and ease of querying through high level languages. 

A relation consists of a relation schema and a relation instance. The relation instance is a table and the relation schema is the column describes the column heads for the table. The schema specifies the relation's name, the name of each field (or column, or attribute), and the domain of each of field. 

```
Students(sid: string, name: string, login: string,
age: integer, gpa: real)
```

So this says that the field named `sid` has a domain named `string`.

An instance of a relation is a set of tuples, also called records, in which each tuple has the same number of fields as the relation schema.
