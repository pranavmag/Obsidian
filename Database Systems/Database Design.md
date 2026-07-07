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