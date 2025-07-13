# MVC vs. DDD

https://leapcell.io/blog/comparing-mvc-and-ddd-layered-architectures-in-go


MVC (Model-View-Controller) is a design pattern mainly used to separate user interface, 
business logic, and data models for easier decoupling and layering, 
 
DDD (Domain-Driven Design) is an architectural methodology aimed at solving design and 
maintenance difficulties in complex systems by building business domain models.


## MVC Structure

| View       | User Interface Layer: responsible for data display and user interaction (such as HTML pages, API responses) |
| :-------- | :----  |
| Controller | Controller Layer: processes user requests, calls Service logic, coordinates Model and View |
| Model      | Model Layer: contains data objects (such as database table structures) and some business logic (often scattered in Service layer) |

## DDD Structure

| User Interface    | Responsible for user interaction and display (such as REST API, Web interface) |
| :---------------- | ---|
| Application Layer | Orchestrates business processes (such as calling domain services, transaction management), does not contain core business rules |
| Domain Layer      | Core business logic layer: contains aggregate roots, entities, value objects, domain services, etc., encapsulates business rules |
| Infrastructure    | Provides technical implementations (such as database access, message queues, external APIs) |

## Main Difference

1. Code Organization Logic

* MVC layers by technical function (Controller/Service/DAO), focusing on technical implementation.
* DDD divides modules by business domain (such as order domain, payment domain), isolating core business logic through bounded contexts.

2. Carrier of Business Logic

* MVC usually adopts an anemic model, separating data (Model) and behavior (Service), which leads to high maintenance cost due to dispersed logic.
* DDD achieves a rich model through aggregate roots and domain services, concentrating business logic in the domain layer and enhancing scalability.

3. Applicability and Cost

* MVC has a low development cost and is suitable for small to medium systems with stable requirements.
* DDD requires upfront domain modeling and a unified language, making it suitable for large systems with complex business and long-term evolution needs, but the team must have domain abstraction capabilities. For example, in e-commerce promotion rules, DDD can prevent logic from being scattered across multiple services.


## DDD

### Layered Architecture
DDD typically adopts a layered architecture. Go projects can follow this structure:

* Domain Layer: Core business logic, e.g., entities and aggregates under the domain directory.
* Application Layer: Use cases and orchestration of business processes.
* Infrastructure Layer: Adapters for database, caching, external APIs, etc.
* Interface Layer: Provides HTTP, gRPC, or CLI interfaces.

### Dependency Inversion
The domain layer should not directly depend on the infrastructure layer; instead, it relies on interfaces for dependency inversion.

Note: The core of DDD architecture is dependency inversion (DIP). The Domain is the innermost core, defining only business rules and interface abstractions. Other layers depend on the Domain for implementation, but the Domain does not depend on any external implementations. In Hexagonal Architecture, the domain layer sits at the core, while other layers (such as application, infrastructure) provide concrete technical details (like database operations, API calls) by implementing interfaces defined by the domain, achieving decoupling between domain and technical implementation.

```go
// Domain layer: defines interface
type UserRepository interface {
    GetByID(id int) (*User, error)
}
```

```go
// Infrastructure layer: database implementation
type userRepositoryImpl struct {
    db *sql.DB
}

func (r *userRepositoryImpl) GetByID(id int) (*User, error) {
    // Database query logic
}
``` 

### Aggregate Management

The aggregate root manages the lifecycle of the entire aggregate:

```go
type Order struct {
    ID      int
    Items   []OrderItem
    Status  string
}

func (o *Order) AddItem(item OrderItem) {
    o.Items = append(o.Items, item)
}
```

### Application Service
Application services encapsulate domain logic, preventing external layers from directly manipulating domain objects:

```go
type OrderService struct {
    repo OrderRepository
}

func (s *OrderService) CreateOrder(userID int, items []OrderItem) (*Order, error) {
    order := Order{UserID: userID, Items: items, Status: "Pending"}
    return s.repo.Save(order)
}
```

### Event-Driven
Domain events are used for decoupling. In Go, you can implement this via Channels or Pub/Sub:

```go
type OrderCreatedEvent struct {
    OrderID int
}

func publishEvent(event OrderCreatedEvent) {
    go func() {
        eventChannel <- event
    }()
}
```

## Summary

### Modularity and Scalability

**MVC**:

* High Coupling: Lacks clear business boundaries; cross-module calls (e.g., order service directly relying on account tables) make code hard to maintain.
* Poor Scalability: Adding new features requires global changes (e.g., adding risk control rules must intrude into order service), easily causing cascading issues.

**DDD**:

* Bounded Context: Modules are divided by business capabilities (e.g., payment domain, risk control domain); event-driven collaboration (e.g., order payment completed event) is used for decoupling.
* Independent Evolution: Each domain module can be upgraded independently (e.g., payment logic optimization does not affect order service), reducing system-level risks.

