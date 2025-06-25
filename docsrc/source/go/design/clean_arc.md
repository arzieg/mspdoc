# Clean Architecture

https://threedots.tech/post/introducing-clean-architecture/

## low coupling, high cohesion

Synonyme: Clean Architecture, Onion Architecture, Hexagonal Architecture, 

* abstracting away implementation details (separation of concerns)

### Separating Ports and Adapters

We take the code in these groups and place it in different packages. We refer to them as “layers”. The layers we usually use are adapters, ports, application, and domain.

* An **adapter** is how your application talks to the external world. You have to adapt your internal structures to what the external API expects. Think SQL queries, HTTP or gRPC clients, file readers and writers, Pub/Sub message publishers.

* A **port** is an input to your application, and the only way the external world can reach it. It could be an HTTP or gRPC server, a CLI command, or a Pub/Sub message subscriber.

* The **application logic** is a thin layer that “glues together” other layers. It’s also known as “use cases”. If you read this code and can’t tell what database it uses or what URL it calls, it’s a good sign. Sometimes it’s very short, and that’s fine. Think about it as an orchestrator.

* If you also follow Domain-Driven Design, you can introduce a domain layer that holds just the business logic.


### The Dependency Inversion Principle

A clear separation between ports, adapters, and application logic is useful by itself. Clean Architecture improves it further with Dependency Inversion. The principle solves the issue of how packages should refer to each other.

The rule states that **outer layers** (implementation details) can refer to inner layers (abstractions), but not the other way around. The inner layers should instead depend on interfaces.

    * The Domain knows nothing about other layers whatsoever. It contains pure business logic.

    * The Application can import domain but knows nothing about outer layers. It has no idea whether it’s being called by an HTTP request, a Pub/Sub handler, or a CLI command.

    * Ports can import inner layers. Ports are the entry points to the application, so they often execute application services or commands. However, they can’t directly access Adapters.

    * Adapters can import inner layers. Usually, they will operate on types found in Application and Domain, for example, retrieving them from the database.

 von Außen nach Innen: Adapters -> Port -> Application -> Domain


TODO: https://threedots.tech/post/basic-cqrs-in-go/
