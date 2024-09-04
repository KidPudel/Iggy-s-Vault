**P2P service for payment handling (supervising) with DDD and event sourcing (for us) for handling money payout or transactions via traders to accounts, and checks that everything is alright** 
**-> … money comes to the trader, and he is making a deal and aggregator gets triggered back.**

  

==**Project components**==

**Services**

- client - Вся логика, связанная с клиентом. Клиентами являются пользователи системы для запросов на выплаты(мерчанты).
- payment-order-projection - проекция заявки. Собирает сущность заявки и сохраняет ее у себя.
- payment-order - Вся логика, связанная с заявками, которую создают клиенты.
- platform - лучше спросить у Арсения или Ильи, но насколько понял отвечает за логику обработок заявок по направлению MARKET - связка направлений на ввод и на вывод(выплаты).
- rating - вся логика с формированием рейтинга трейдера, на основе которого формируется логика назначения заявок в сервисе scheduler.
- receipt-analysis - валидация чеков.
- receipt - Вся логика с чеками(загрузка, получение, прикрепление к заявке и запрос на валидацию)
- Receipt-projection - проекция самого чека. Собирает сущность чека и сохраняет ее в себя.
- trader-projection - проекция трейедера. Собирает сущность заявки и сохраняет ее у себя.
- trader - вся логика, связанная с трейдером. Трейдерами являются пользователи системы для осуществления саммих выплат по заявкам.
- withdrawal-order - Вся логика по выплатам вознаграждений трейдерам.
- withdrawal-order-projection - проекция. Собирает сущность заявки и сохраняет ее у себя.

**Other**
- Templates
- Real DDD template to utilize
- Tools
- Future Framework (present pkg)
- UI
- api-gateway - Обратное проксирование. Все запросы идут через него, он проксирует их во внутреннюю систему с формированием правильной авторизации, так как наш проект состоит из 2 независимых комманд(Onlyfinance)
- Reverse proxy / authorization substitution 
- CI
- Continues integration (for real)
- event-aggregator - получает евенты заявки и трейдера и сохраняет их в clickhouse. Нужен для агрегации проекций заявок для анализа и формирования рейтинга в сервисе rating. 
- Aggregating orders to the clickhouse (analyze) for composing trader’s rating 
- scheduler - Вся логика, которая автоматизирована. Например назначение заявок, перевод статусов, пополения балансов и так далее.
- фоновые процессы like giving money to trader
- Handles payment orders statuses, deadlines (assigned, in progress)
- Describes holding balance for the trader (sends regularly request to the trader)
- (Soon) checks orders that has status “in review”, and forcing it for review receipts
- Assigns orders, traverse through orders, checks all available traders, takes on account a lot of limitations (traders and itself) 
- Test-migration
  

==**Architecture**==
- Payloads - what sends/write from it (data being transmitted with the event)
- EventKinds - defines events happens to this domain / entity
- Controller: HTTP handler

- models - data model that is retrieved 

- Queue controllers: (wrapper of consumer)  which could be subscribed/listen to many services! (Kafka defined in main, which services listen to which)
- Projection: is the _projection_ of the necessary part of an entity that is composed via grabbing its history of events
- We define events (kinds and payloads (what we send with event)) for the entities and then utilizing it in processor of the events where we map controller models to the payload event models and generating an event
- Applier is processing events
- Processor, generates events based on commands
- Usecase **(which is the wrapper of base usecase)** , is doing all business logic, but in services we are **_exposing_** use case for init and _preprocessing_ commands (before processing) (that are going from controller) to or fill the command params, by getting entity and checking with the logic

- In projection though, in use case we just create and update from params

- Event payloads, is the payloads we’ve got from the request, that to the event payload in queue control, that gone to command, to the event



==**Flow**==
**http, kafka(if event from service)**

**1-> Controller: catch requests and map it to the command**

**1-> queue-controller: catch event from other service and map it to the command**

- we only map to the commands only for main entities, for projection we Map to usecase params

**2-> use case: call base usecase and pre-processor of commands** 

**2-> base usecase: handle command (what is happening internally)  
**- construct/get current entity **based on current/up to date events**

- generate new events based on command, and entity info

- after we’ve got new events, alter an entity with them for the response

- and write those events to DB

**3-> processor: based on commands and constructed entity from DB, form/generate events**

**_4->_** **applier: based on event changing entities, for the response**

**5-> repository: add new events to DB, to update entity history (event sourcing)**

- we only store main entities via sqrs (event sourcing), projection we store as normally we would 

**6->  event publisher: to send to the queue (wrapper of producer): generated events, we handle, and based on that, we send payloadModels corresponding to that events**

  

Base use case:

- Processor
- Applier
- Repository

Entities:

- Entity type itself
- Entity builder
- Value types: complex types of entity
- **Exposing** events that they catch from others and events that they produce, _could be the same_
- Commands that they handle to form events to record to db
- Processor: process commands to form events
- Applier: based on events, edit entity, that will be recorded to db

Repository:

функцию TransformEventPayloadModelToEventPayload(), преобразующую модели репозитория для payload событий в эти самые payload: to transform repository payload to the existing events

функцию TransformEventToPayloadModel(), преобразующую payload событий в их модели репозитория: to write events to the repository