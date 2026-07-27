2026-07-13 19:17

Tags: 

The book does actually go into SQL but I will not be including it into my notes because I have a working knowledge of SQL, I will just read the part. The parts I include here will be terminology I'm hearing for the first time in the context of DBMS.

A relational database is a collection of relations, each relation being a table with rows and columns. This allows for a simple data representation and ease of querying through high level languages. 

A relation consists of a relation schema and a relation instance. The relation instance is a table and the relation schema is the column describes the column heads for the table. The schema specifies the relation's name, the name of each field (or column, or attribute), and the domain of each of field. 

```
Students(sid: string, name: string, login: string,
age: integer, gpa: real)
```

So this says that the field named `sid` has a domain named `string`.

An instance of a relation is a set of tuples, also called records, in which each tuple has the same number of fields as the relation schema. So each tuple is a row and each column is a field. So if we have a Students table with columns named `sid, name, email, age, gpa` then each of those are fields and each row made for one student would be a tuple. Each relation must be a set of unique tuples or rows, no two rows can be identical. This makes sense to me because no two students can have the exact same fields because then they cannot be uniquely identified. 

So the table is the relation instance, the rows are the tuples (records), and the columns are fields (attributes). That's the terminology it seems like we will be using throughout.

Also the degree (arity) of a relation is the number of fields and the cardinality of a relation is the number of tuples in it.

#### Domains

If relation R has n fields, and each field f is associated with a domain name D (like "integer", "string", "date", etc.) then the notation would look something like this:

`{ (f1​:d1​ ,…, fn​:dn​) ∣ d1 ​∈ Dom(D1​) ,…, dn ​∈ Dom(Dn​) }`

An instance of R has a set of n-tuples, where in each individual tuple, the value in position i (call it di) must be drawn from the domain set associated with field i.

From what I understand, this means that if we have an tuple like 
`(sid: "53666", name: "Nav", gpa: 3.4)` then the value `"53666"` must come from `Dom(string)`, the value `"Nav"` must come from `Dom(string)`, and the value `3.4` must come from `Dom(real)`.

As you can see there are three independent domain checks as stated in the notation with `d1 ​∈ Dom(D1​) ,…, dn ​∈ Dom(Dn​)`.

`d1` is a placeholder for whatever value sits in the first field (f1) of a tuple, and it must come from `Dom(D1)`, the domain which is assigned to that field, in our case it is `Dom(String)`. So every `sid` value takes from `Dom(String)`, so on and so forth with the other fields until we reach the end of `dn`, which in our case the last one is for the `gpa` field.

#### Integrity Constraints

Databases need to make sure that the data it stores is valid data, so we can use integrity constraints. An integrity constraint is a condition specified on a database schema for the data allowed to be stored. If a database instance satisfies all integrity constraints specified then it is a legal instance. A DBMS enforces these integrity constraints so that only legal instances can be stored in the database.

Example integrity constraints might be like not allowing two students to have the same `sid` or only allowing positive integer values in the `age` field.

When the database schema is defined, integrity constraints must be specified and when a database application is run the DBMS checks for violations and disallows those specific changes or make some compensating changes to the data to satisfy the integrity constraints.

##### Key Constraint

A key constraint is a statement that a certain minimal subset of the fields of a relation is a unique identifier for a tuple. Not allowing two students to have the same `sid` would be an example of a key constraint. These set of fields that uniquely identify a tuple based off the key constraint is known as a candidate key for the relation. In our example, we can say that `sid` is a candidate key.

We have to be careful with what we make a key constraint. I think it's reasonable for a student id to be a key constraint because it's possible to make it so that no two students have the same one. But, having a student name as a key constraint would not work because people can have the same names. The set of fields associated with a candidate key needs to be able to uniquely identify a tuple. 

However, `{sid, name}` is not a candidate key. That would be considered a superkey, which is a set of fields that contains a candidate key. Candidate keys need to be minimal, if you can uniquely identify a student using `sid` then do not make a relation have several candidate keys.

A relation can have several candidate keys or several fields as candidate keys though. For example if we made the key `{login, age}` it would then mean that users can have either the same login or age but not both. From what I understand, if one field is not enough to uniquely identify then you may use multiple fields as a candidate key. It does seem a bit tricky because in most circumstances `login` would be enough to be a candidate key but I guess it is up to the people who design the database.

##### Primary Key

Out of the candidate keys, we choose a primary key. I was a bit confused by this, because if we have candidate keys that uniquely identify tuples then why would we need a primary key? It seems that this is because a primary key is the official unique identifier that other tables will link to using a foreign key and that the system will optimize for.

```
CREATE TABLE Students ( sid CHAR(20) ,
 name CHAR (30) ,
  login CHAR(20) , 
  age INTEGER, gpa REAL, 
  UNIQUE (name, age), 
  CONSTRAINT StudentsKey PRIMARY KEY (sid) )
```

From what I understand, `{name, age}` and `{sid}` can be used as candidate keys, meaning that they can both be used to uniquely identify a tuple but only `{sid}` is the primary key that can allow for other tables to link to it using a foreign key, automatic indexing, and clustering.

##### Foreign Key Constraint

Sometimes the information stored in a relation is linked to the information stored in another relation. The common integrity constraint that involves two relations is a foreign key constraint.

Lets say we have a second relation in addition to our Students one from earlier called Enrolled:

`Enrolled(studid: string, cid: string, grade: string)`

The `studid` field of `Enrolled` is called a foreign key and refers to `Students`. The foreign key in the referencing relation must match the primary key of the referenced relation. So the primary key and foreign key must match in the number of columns and the data types, but as we see in the example above the name does not have to match.

So if we try to insert a tuple with a `studid` into `Enrolled` where the corresponding `sid` in `Students` does not exist, then the integrity constraint is violated and the DBMS should reject the insertion. Also if we deleted a tuple from `Students` that a tuple from `Enrolled` was linked to, the DBMS should disallow it or also delete the tuple from `Enrolled`.

Null values are allowed in a foreign key and don't violate the constraint, but a primary key cannot be a null value because of its inherent nature to uniquely identify a tuple.

#### General Constraint

We can add even more constraints such as adding the constraint that all students in the table must have a GPA > 2.0. This would be known as a domain constraint because it defines the set of permissible `GPA` values more strictly than is possible by simply using a standard domain such as only allowing `Real` values. Current relational database systems support such general constraints in the form of table constraints and assertions.

Table constraints are associated with a single table and checked whenever that table is modified and assertions involve several tables and are checked whenever any of those tables are modified.















