2026-08-11 21:26

Tags:

### Tuple Relational Calculus

A tuple variable is a variable that takes on tuples of a particular relation schema as its values and each value of a tuple variable has the same number and type of fields. A tuple relational calculus query has the form `{ T | p(T) }`, where T is a tuple variable and p(T) is a formula that describes T. The result of this query is the set of all tuples t for which the formula p(T) evaluates to true with T = t.

For example, `Find all sailors with a rating above 7.` The query would be

`{ S | S E Sailors AND S.rating > 7 }`

Lets say that the 'AND' is replaced by the proper symbol.

It should translate to `Return the set of tuples S such that S is in the Sailors relation AND the rating of S is greater than 7.`

#### Formulas

An atomic formula is the most basic, indivisible true/false statement in relational calculus and it cannot be broken down into smaller formulas. It consists of checking if a tuple belongs to a relation `(R E Rel)`, comparing an attribute of one tuple to an attribute of another `(R.a op S.b)`, or comparing an attribute to a static value `(R.a op constant), (constant op R.a)`.

A general formula can be built by chaining together smaller formulas. They can be just atomic formulas or complex formulas. Complex formulas take smaller formulas and combine them using logical operators like "AND" or "NOT", or wrap them in quantifiers like "exists" or "for all".

#### Bound vs Free Variables

When a variable has a quantifier, it is bound to that quantifier. The quantifier controls the variable entirely meaning that the statement is completely self contained and the database can evaluate this as true or false all on its own by scanning the table.

A variable is free if it has no quantifier attached like `S.rating > 7`. Due to S being a free variable, the statement is incomplete on its own meaning that the database cannot evaluate it as true or false until a specific record is handed to it from the outside to plug into the S placeholder.

If we say like `There exists an R where R.age > 20`, that has the "exists" quantifier attached to it and we can reasonably evaluate it as true of false without needing a specific record to replace R whereas we can't say the same for `S.rating > 7` because we need a specific record to determine whether it satisfies the condition or not. So if we pass in a record `{'Bob', 8, 3.2}`, then we can evaluate it as true.




