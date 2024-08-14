**P2P service for payment handling (supervising) with DDD and event sourcing (for us) for handling money payout or transactions via traders to accounts, and checks that everything is alright** 

**-> … money comes to the trader, and he is making a deal and aggregator gets triggered back**

  
We have clients, traders, payment orders (заявки), receipts (чеки)

  

## ==**Project components**==

Services

- Client-serivice
- Payment-order-projection-sercice
- Payment-order-service
- Platform-service 
- Rating-service
- Receipt-analysis-service

- Validation

- Receipt-service
- Trader-projection-serviec
- Trader-service

- Took order
- Loading something
- Login
- Transfer money to the trader

  

Other

- Templates

- Real DDD template to utilize

- Tools

- Future Frameworks (present pkg)

- UI
- Api-gateway

- Reverse proxy / authorization substitution 

- CI

- Continues integration (for real)

- Event-aggregator

- Aggregating orders to the clickhouse (analyze) for composing trader’s rating 

- Scheduler

- Handles payment orders statuses, deadlines (assigned, in progress)
- Describes holding balance for the trader (sends regularly request to the trader)
- (Soon) checks orders that has status “in review”, and forcing it for review receipts
- Assigns orders, traverse through orders, checks all available traders, takes on account a lot of limitations (traders and itself) 

- Test-migration

  

  

  

## ==**Architecture**==

  

- Payloads - what sends/write from it (data being transmitted with the event)
- EventKinds - defines events happens to this domain / entity
- Controller/models - data model that is retrieved 
- Queue controllers - from kafka
- Projection: is the _projection_ of the necessary part of an entity that is composed via grabbing its history of events
- We define events (kinds and payloads (what we send with event)) for the entities and then utilizing it in processor of the events where we map controller models to the payload event models and generating an event
- Applier is processing events
- Processor, generates events based on commands
- Usecase, is doing all business logic, but in services we are **_exposing_** use case for init and _preprocessing_ commands (before processing) (that are going from controller) to or fill the command params, by getting entity and checking with the logic

  

  

## ==**Flow**==

  

**(http, kafka)Controller -> use case -> processor -> applier -> usecase (the rest of business logic) | or event publisher to send to the queue**

Some of the processes
**Queue controller via event generates commands, ran thoguh ProcessCommand in processor.go and in switch do something or generates event**
method_params.go is for mapping from json to type
Request comes to controller, mapping is happening (happening at each layer), (we generate commands from events, in critical/central entities), in command processing useceases are triggered

**Event sourcing** - entity in progress of events (change name, birthday) this event recorder in db (in every event id of entities of course  
  
change entity, take all events, apply - **compose**

  

Pub to db and put to kafka pub sub

  

projection: composing, отражать текущую сущность with specific values service need and takes it from kafka


Для трейдоров заявки Обмен валют на счита

**For traders, orders is call for money exchange on accounts**


**Applier apply changes and validates entities** 

**Scheduler - фоновые процессы like giving money to trader**


**Mapping from transforming model entity to other event**
P2p review for passing links on pull request

P2p it - questions
—— event sourcing with ddd
В каждом сервисе проекция конкретной сущности существует лишь в том виде, в котором она нужна этому сервису. До этого момента шедулеру не требовался ID чека, поэтому его и нет в его проекции заявки.
Однако есть сервис, имеющий наиболее полную проекцию заявки. Это сервис "Заявка/Проекция": [https://git.bwg-io.site/waterfall/payment-order-projection-service](https://git.bwg-io.site/waterfall/payment-order-projection-service) **Т.к. ему нужно отображать всю информацию по заявке, он и сохраняет её всю.**