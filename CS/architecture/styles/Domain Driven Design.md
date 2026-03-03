In DDD focus in the entire **domain model**.

**Domain**: Specific business or application area that software system is being designed to address
**Domain model**: Conceptual model of the business domain, representing the key concepts, their attributes, behavior, and relationship. In essence, the structured representation of the knowledge and process within a specific business area

Example of domain model:
E-commerce Domain Model:
- Entities: Product, Customers, Order, Shopping cart
- Value Objections: Money, Address, Product Review
- Aggregates: Order (root) with OrderItems
- Domain Services: PricingService, InventoryService


# Aggregate
**Aggregate is a cluster of *related* objects/entities by business logic** that are treated as a single unit for data changes (encapsulates business rules and logic for consistency)

Aggregate has a root object, known as an **aggregate root**, which serves as an ***entry point for all operations on the aggregate***

Example:
- We have a **route** of **locations** at which one of them is a **vending machine**
And we can assign our route as a **root**, so that we can via route define locations and add vending machines
- Mom gives as 50$ to clean dishes, we aggregate it to the younger brother to do this for 25$, and so on..

> It is recommended to organize ***event streams per aggregate*** ([[event sourcing]]), usually identified by the id of the aggregate root

![[Pasted image 20240704163555.png]]

