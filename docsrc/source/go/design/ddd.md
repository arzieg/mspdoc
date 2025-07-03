# Domain Driven Design

https://programmingpercy.tech/blog/how-to-domain-driven-design-ddd-golang/


Domain-Driven Design is a way of structuring and modeling the software after the Domain it belongs to. The domain is the topic or problem that the software intends to work on. The software should be written to reflect the domain.

Domain := The domain is the area that the software will be operating in, I would call the Tavern the Core/Root domain.

Subdomain := We have also gotten a few Sub-domains which are the things top hat mentions that the tavern needs. A subdomain is a separate domain used to solve an area inside the root domain.

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

# Abstraktion

Abstraktion wird auf den Aggregaten erzeugt. Hier werden Design Pattern angewandt. Z.B. neuer Customer, ein FactoryPattern welches einen Pointer zu einem neuen Customer zurückgibt. 

## Repository pattern

DDD describes that repositories should be used to store and manage aggregates. It is a pattern that relies on hiding the implementation of the storage/database solution behind an interface. This allows us to define a set of methods that has to be present, and if they are present it is qualified to be used as a repository.

The advantage of this design pattern is that it allows us to exchange the solution without breaking anything.

In aggregates liegt die Busines Logic. Dort wird eine repository.go angelegt

```
type CustomerRepository interface {
	Get(uuid.UUID) (aggregate.Customer, error)
	Add(aggregate.Customer) error
	Update(aggregate.Customer) error
}
``` 

mit einem Interface, d.h. den benötigten Funktionen, die ein anderes go-modul bedienen muss. 

Bsp. MemoryDatenbank, in einem eigenen Package werden dann die Funktionen geschrieben, die das CustomerRepository interface "bedienen" können, dh. dort gibt es dann auch Get, Add und Update Methoden(Funktionen)

## Service

A service will tie all loosely coupled repositories into a business logic that fulfills the needs of a certain domain. 
A service typically holds all the repositories needed to perform a certain business logic flow, such as an Order, Api, or Billing. What’s great is that you can even have a service inside a service.






