# Domain Driven Design

https://programmingpercy.tech/blog/how-to-domain-driven-design-ddd-golang/


Domain-Driven Design is a way of structuring and modeling the software after the Domain it belongs to. The domain is the topic or problem that the software intends to work on. The software should be written to reflect the domain.

Entity := Unique identifier, mutable

    An entity is a struct that has an Identifier and that can change state, by changing state we mean that the values of the entity can change.
    We will create two entities, to begin with, Person and Item. I do like to keep my entities in a separate package so that they can be used by all other domains.

Value Object := no identifier, immutable

    There can be occurrences where we have structs that are immutable and do not need a unique identifier, these structs are called Value Objects. Value objects are often found inside domains and used to describe certain aspects in that domain. We will be creating one value object for now which is Transaction, once a transaction is performed, it cannot change state.

Aggregates := unique identifier by root entry, multiple entries/value objects combined

    root entry (person) has a product (entity) and a transaction (value object)
    An Aggregate is a set of entities and value objects combined. So in our case, we can begin by creating a new aggregate which is Customer.
    The reason for an aggregate is that the business logic will be applied on the Customer aggregate, instead of each Entity holding the logic. An aggregate does not allow direct access to underlying entities.

    An important rule in DDD aggregates is that they should only have one entity act as a root entity. What this means is that the reference of the root entity is also used to reference the aggregate. For our customer aggregate, this means that the Person ID is the unique identifier.

    Notice that all fields in the struct begins with lower case letters, this is a way in Go to make an object inaccessible from outside of the package the struct is defined in. This is done because an Aggregate should not allow direct access to the data. Neither does the struct define any tags for how the data is formatted such as json.

    I set all the entities as pointers, this is because an entity can change state and I want that to reflect across all instances of the runtime that has access to it. The value objects are held as nonpointers though since they cannot change state.

    






