Approach of **storing/recording a data as an events/actions as the point of truth** describing the state of a business entity resulting in a collection of events, called "***event stream***", which is kind of a "journey" of an entity that *tells how and why (what happened) we get there*.
Those events are stored per aggregate
So we can use our events to derive to current state (or any point of time)

So event sourcing is opposite to the [[CRUD]] approach where we store our current state of the records. Which results in us not knowing how we got to our result (could be achieved with logs), and why (what happened)
- Event sourcing allows: Create, Read
- But it forbids: Update, Delete. Because it's a destructive operations, that alter the timeline of even

**Event source system** reading all of the events from the client and performing the computations that results in a client state represented in a regular table.
![[Pasted image 20240703164827.png]]


# events
**Event is the fact**, that cannot be retracted in other way
or in [[Domain Driven Design]]
**Event is a significant *domain-specific* occurrence**
Difference is that:
- In EDD events are more granular (ItemAdded, CartTotalUpdated)
- In DDD more focused on specific to domain-relevantgi significance (ItemAddedToCart)
## Fun example:
The fight result is a fact. It cannot be retracted. It happened on 25th May 1965, so at a certain point in time. It has specific information about what has happened, like information of:

- who fought,
- that it only lasted 1 minute 44 seconds,
- the venue was Central Maine Youth Center in Lewiston, Maine.

## Technical example:
User added the item to the basket

# projections
**Projection is the events that triggered by event/fact**, could be interpreted differently
or in [[Domain Driven Design]]
**Projection is the event that triggered by center event/fact** that is more tightly coupled to the center domain model event
Domain data that is optimized for querying


## Fun example:
After the event, fans speculate whether the fight was real

## Technical example:
User added the item to the basket, so view got updated, change the product availability


# aggregates
**Aggregate is a cluster of *related* objects/entities by business logic** that are treated as a single unit for data changes (for data consistency purpose)

Aggregate has a root object, known as an **aggregate root**, which serves as an ***entry point for all operations on the aggregate***

Example:
We have a **route** of **locations** at which one of them is a **vending machine**
And we can assign our route as a **root**, so that we can via route define locations and add vending machines

> It is recommended to organize ***event streams per aggregate***, usually identified by the id of the aggregate root

![[Pasted image 20240704163555.png]]

