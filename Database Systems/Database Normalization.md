2026-07-26 18:33

Tags: 
#### First Normal Form

First Normal Form Rules:
1. A table without a primary key is not allowed
2. Using row order to convey information is not allowed
3. Mixing data types within the same column is not allowed
4. Repeating groups is not allowed

#### Second Normal Form

Deletion Anomaly: An anomaly that occurs when the deletion of record results in the loss of other unrelated data.

Update Anomaly: An anomaly that occurs in bad database designs when updating a certain record but you needed to update multiple records to maintain accuracy. A partial update would lead to inconsistent data meaning you have an update anomaly.

Insertion Anomaly: An anomaly that occurs when you want to insert information but either primary key is not known or other unrelated data also needs to be filled that you may not have access to or are unable to input at the moment. The insertion will fail and the data you wanted to record goes unrecorded.

In second normal form, each non-key attribute must depend on the entire primary key.

For example if we have the primary key as `{Player_ID, Item_Type}` and our bad database design had two other fields `Item_Quantity` and `Player_Rating` we have to see if both of these fields actually depend on the entire primary key. `Item_Quantity` does depend on the entire primary key because it describes the amount of a certain item a player has, however `Player_Rating` only needs to know the `Player_ID` as it has nothing to do with items. So, `Player_Rating` does not need to full primary key of this table and violates second normal form. To fix this we can create a separate table for `Player_Rating` where the primary key is `Player_ID`.

#### Third Normal Form

You may have instances where a attribute depends on the key but only indirectly. For example if we have a table that has fields `Player_ID`, `Player_Rating`, and `Player_Skill_Level` and the rating depends on what their skill value is. This is called a transitive dependency. 

The dependencies would be:

`{Player_ID}` -> `{Player_Skill_Level}`

`{Player_ID}` -> `{Player_Skill_Level}` -> `{Player_Rating}`

This violates third normal form which states that every non-key attribute in a table should depend on the key, the whole key, and nothing but the key. A non-key attribute cannot depend on another non-key attribute.

A slightly stricter version of third normal form would be Boyce-Codd Normal Form which is that every ~~non-key~~ attribute in a table should depend on the key, the whole key, and nothing but the key. There are situations where you may have multiple candidate keys that can uniquely identify records so this stricter rule eliminates all non-superkey dependencies, the super key being the primary key. The relationship would be `X -> Y`, in which `X` is the superkey.

#### Fourth Normal Form

Fourth normal form states that multivalued dependencies in a table must be multivalued dependencies on the key.

A multivalued dependency occurs when an attribute determines multiple independent values of another attribute, regardless of other columns. For example in a table that has `Courses`, `Instructors`, and `Textbooks`, a course could have multiple instructors and textbooks, but instructors and textbooks are independent of each other. This can create redundant combinations where every instructor is paired with every textbook for that course.

Split the table into two tables consisting of `Courses` and `Instructors` and `Courses` and `Textbooks`.

#### Fifth Normal Form

Fifth Normal Form states that the table (which must be in 4NF) cannot be describable as the logical result of joining some other tables together.

For example a table recording which `Agents` sell which `Products` for which `Companies` may violate 5NF if the combination of agent, product, and company is governed by a complex rule rather than just the primary key. Put these into separate tables like agent-company, company-product, and agent-product would eliminate the redundancy and preserve all original data integrity.
