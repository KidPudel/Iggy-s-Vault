**Waterfall**

**P2P service for OTC aggregator for payment gateway with DDD for handling money transactions via traders to accounts**

**Client (business), gives targets (cards), and puts money on, for** 

  

Utilizes Yandex OCR for computer vision which accomplishes text recognition in files like PDF and images

  

We have clients, traders, payment orders (заявки), receipts (чеки)

  

**Project components**

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

  

  

  

  

  

————————————————————

  

Client machines for receipts (заявки)

  

Trader - closing recepts, get money

  

  

**Controller handles request from user**

  

**Queue controller via event generates commands, ran thoguh ProcessCommand in processor.go and in switch do something or generates event**

  

method_params.go is for mapping from json to type

  

Request comes to controller, mapping is happening (happening at each layer), (we generate commands from events, in critical/central entities), in command processing useceases are triggered

  

  

Business logic without  …

  

Use cases with …

  

Repository 

  

  

Dd domain driven delevekemnt

  

Domain - entity (trader, receipt, racket)

  

All bound to domain like receipt 

  

  

——————————————————————————————————————————————

  

**Event sourcing** - entity in progress of events (change name, birthday) this event recorder in db (in every event id of entities of course  
  
change entity, take all events, apply - **compose**

  

Pub to db and put to kafka pub sub

  

projection: composing, отражать текущую сущность with specific values service need and takes it from kafka

  

**Projection: is the _projection_ of the necessary part of an entity that is composed via grabbing its history of events**

  

Each service can store projection of entity via subscription

  

————

  

  

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