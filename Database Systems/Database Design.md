2026-07-02 19:57

Tags: 
### Entities, Attributes, and Entity Sets

An entity represents a specific object that can be distinguishable from other objects in the same entity set. The entity set is the overall collection of objects. Each entity in the entity set must have the same attributes.

For example an entity can be a specific dinosaur toy, the entity set will be the whole collection of all toys, and the attributes can be like `price`, `quantity`, and `toy_id`. Each toy will have these same attributes that can be referenced. 

Each attribute within an entity set we also have to identify a domain, which is a list of possible values. This is easy to remember because in math we were always taught about domains being this way. For example the attribute `price` may be represented by an int or a double. 

We also have a key to go with each specific entity in the entity set. A key is a minimal set of attributes whose values uniquely identify an entity in the set. For example we want each toy to be identified by their unique `toy_id` so we can make that its primary key.

```
// Entity
Entity {
toy_id: 1,
price: 19.99
quantity: 5
}

// Entity Set
EntitySet<Toy> = { Entity {. . .}, Entity {. . .}}

// Attribute
Entity.toy_id

// Domain
Type: f64

// Key
Primary Key(toy_id)
```

### Relationships and Relationship Sets

A relationship is an association among two or more entities. For example "Nav works in the Database Department", has the two entities `Nav` (Employee) and `Database Department` (Department) and the connection is the WORKS_IN relationship.

Just like an entity set that is the collection of similar entities, a Relationship Set can be a collection of similar relationships.

A relationship can also have descriptive attributes which is information about the relationship. If we want to say, "Nav works in the Database Department as of 2027" we can add the attribute `since` to `Works_In`

![[Pasted image 20260708224836.png]]

We can set key constraints here that can restrict the relationship of fields to be a certain way. A relationship set like `Manages` is able to be one-to-many meaning that one employee can be associated with many departments as a manager whereas each department can only be associated with one employee as its manager. But a relationship set like the `Works_In` relationship set can be many-to-many because an employee is allowed to work in several departments and a department can have several employees. We can also set a restriction that each employee can manage at most one department to the Manages relationship set and then it would be a one-to-one relationship set.

There is also a type of constraint called a participation constraint which requires a relationship to occur. For example, if every department is required to have a manager then that is a participation constraint.

There can be total participation or partial participation. For the previous example it is total participation since it is for every department, but partial participation would be like `Employees` in `Manages` since not every employee gets to manage a department.

### Weak Entities

A weak entity is an entity that can only exist and be uniquely identified with the primary key of another entity called its identifying owner. If the identifying owner is deleted then all weak entities associated with it are also deleted. For example we have employees that have insurance policies, the insurance policies that an employee has can be our weak entities that can be uniquely identified by the employee's ssn. If an employee quits then their policies are automatically deleted along with the employee information.

The owner entity set and the weak entity set must participate in a one-to-many relationship because one owner entity can have many weak entities associated with it. The weak entity set must also have total participation int he identifying relationship set because it cannot exist without an identifying owner.

The attributes of a weak entity set that uniquely identify a weak entity for a given owner entity is called a partial key of the weak entity set. 

### Class Hierarchies

Class Hierarchies allow entities from an entity set to be classified into subclasses. For example if we have employees that can be paid by hourly wage or paid by contract we can derive subclasses called hourly_emps and contract_emps which will both be entity sets. These entity sets will have all the attributes of the employees entity set that they inherited from.

Usually these are represented by IS A relationships. So hourly_emps IS A employees, would be the way to read that.

A class hierarchy can be viewed in one of two ways. We can have Generalization which is where the common characteristics of multiple entity sets are used to make a new entity set that has these shared characteristics. So this means that first the subclasses are made and the superclasses are made from those common shared characteristics in each subclass. The other way to look at it is Specialization which is where the superclasses are made first and the subclasses and their specific attributes are made after.

We can also specify two kinds of constraints for these: overlap constraints and covering constraints. Overlap constraints determine whether two subclasses can contain the same entity. For example can an employee be both an hourly_emps and a contract_emps? Most likely not, but if we define more subclasses like senior_emps and junior_emps, then a employee who is an hourly_emps can also be a senior_emps or junior_emps. We can denote that with `contracts_emps OVERLAPS senior_emps`. If we don't have a statement like this, the default will be that the entity sets are constrained to have no overlap.

Covering constraints determine whether the entities in the subclasses collectively include all entities in the superclass. For example, does every employees entity have to belong to one of its subclasses? Probably not, but if we say that an employee can only be a junior_emp or a senior_emp then we can say that `senior_emps AND junior_emps COVER employees`. If we don't have a statement like this, the default will be that the entity sets have no covering constraints, so we can have employees that aren't senior_emps or junior_emps.



