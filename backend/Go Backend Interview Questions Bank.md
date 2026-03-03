# Go Backend Interview Questions Bank

> This is what interviewers actually ask. Not textbook definitions — real question chains with follow-ups that test depth.
>
> How to use: pick a section, cover the answers, try to answer out loud. The follow-up questions are what separate "I read about it" from "I understand it." If you can handle the follow-ups, you're ready.
>
> Related:
> - [[Go Backend Deep Learning Method]] — how to learn
> - [[Go Backend Interview Preparation]] — reference to look things up

---

# Round 1 — Go Language

## Q1: Slices vs Arrays
**"What's the difference between a slice and an array in Go?"**

Follow-ups:
- What does the slice header contain internally?
- If I do `b := a[1:3]` and then modify `b[0]`, does `a` change? Why?
- What happens when you `append` to a slice that's at capacity?
- How does the growth strategy work? Does it always double?
- If I pass a slice to a function and append inside the function, does the caller see the new elements? Why or why not?
- When would you use an array instead of a slice?

*What they're testing: do you understand value vs reference semantics, backing array sharing, and the implications for concurrent code.*

---

## Q2: Interfaces
**"How do interfaces work in Go? How is this different from, say, Java or C#?"**

Follow-ups:
- What does an interface look like in memory? What are its two fields?
- Why does `var i error = (*MyError)(nil)` make `i != nil` true?
- How would you check the concrete type stored in an interface at runtime?
- What's the difference between a type assertion and a type switch?
- Can you give me an example of interface composition in the standard library?
- What does "accept interfaces, return structs" mean? Why is it idiomatic?
- When would you define an interface at the consumer side vs the producer side?

*What they're testing: implicit satisfaction, nil interface trap (very common gotcha), and whether you design with interfaces idiomatically.*

---

## Q3: Error Handling
**"How does Go handle errors? Walk me through your approach."**

Follow-ups:
- Why doesn't Go have exceptions?
- What does `%w` do in `fmt.Errorf` and why is it important?
- What's the difference between `errors.Is` and `errors.As`? Give me a real example.
- When would you create a custom error type vs using a sentinel error?
- How do you decide what to wrap and what not to wrap?
- What is `panic`? When is it acceptable to use in production code?
- How does `recover` work? Where must it be called?
- You have a service that calls 3 external APIs. How do you handle partial failures?

*What they're testing: do you handle errors thoughtfully or just `if err != nil { return err }` everywhere.*

---

## Q4: Goroutines
**"Explain goroutines. How are they different from OS threads?"**

Follow-ups:
- How much memory does a goroutine use initially? How does the stack grow?
- Explain the GMP model. What is G, M, and P?
- What is GOMAXPROCS? What happens if you set it to 1?
- If a goroutine makes a blocking syscall, what happens to the M and P?
- How would you detect a goroutine leak? What causes them?
- You launched 100,000 goroutines. Is this a problem? When does it become one?
- How does `go run -race` work? What does it detect and what does it miss?

*What they're testing: do you understand the runtime machinery or do you just use `go func()` without thinking.*

---

## Q5: Channels
**"Explain channels. When would you use a channel vs a mutex?"**

Follow-ups:
- What happens if you send on a nil channel? Receive from a nil channel? Close a nil channel?
- What happens if you send on a closed channel? Receive from a closed channel?
- Buffered vs unbuffered — what's the semantic difference? When do you choose each?
- How does `select` work? What happens if multiple cases are ready simultaneously?
- Show me the pattern for using a channel as a semaphore to limit concurrency.
- How do you signal cancellation to multiple goroutines? (hint: close a channel or context)
- What's the rule about who should close a channel — sender or receiver? Why?

*What they're testing: channel axioms (nil/closed behavior), and whether you can reason about deadlocks.*

---

## Q6: Context
**"What is `context.Context` and how do you use it?"**

Follow-ups:
- What are the different ways to create a context? When do you use each?
- What happens if you forget to call `cancel()` on a context with timeout?
- Should you store a context in a struct? Why or why not?
- How do you propagate request-scoped values (like request ID) through your application?
- What are the pitfalls of `context.WithValue`?
- Your database query is running. The client disconnects. How does context cancellation reach the DB driver?
- How is context used in your HTTP middleware chain?

*What they're testing: do you understand context lifecycle, cancellation propagation, and common anti-patterns.*

---

## Q7: Maps
**"Tell me about maps in Go."**

Follow-ups:
- What is the zero value of a map? What happens if you write to it?
- Are maps safe for concurrent use? What happens if two goroutines write to the same map?
- What is `sync.Map`? When would you use it instead of a regular map with a mutex?
- How does a map work internally? (buckets, hash, growth)
- Why is map iteration order random?
- If you delete from a map inside a range loop, is that safe?
- How would you implement a concurrent-safe map with sharding for better performance than a single mutex?

*What they're testing: concurrency safety awareness, understanding of zero values, internal structure.*

---

## Q8: Generics
**"Go added generics in 1.18. How do you feel about them? When do you use them?"**

Follow-ups:
- What's the difference between `any` and a specific type constraint?
- What is the `~` (tilde) in a type constraint?
- Can you have generic methods on a struct? (No — only generic functions or generic types)
- When would you use generics vs using an interface?
- Show me a real-world example where generics improve code (Map, Filter, Result type, etc.)
- What are the limitations of Go generics compared to, say, Rust or TypeScript?

*What they're testing: practical judgment — do you use generics where they help or over-engineer with them.*

---

## Q9: Defer
**"How does `defer` work? What's the execution order?"**

Follow-ups:
- Deferred functions execute in what order? (LIFO)
- When are defer arguments evaluated — at the defer statement or when the function returns?
- What's a common bug with defer inside a loop?
- How do you use defer for cleanup (file handles, DB connections, mutex unlock)?
- What happens if a deferred function panics?
- Can you modify return values from a deferred function? (yes, with named returns)

*What they're testing: subtle behavior — argument evaluation timing and interaction with named returns.*

---

## Q10: Memory and GC
**"How does garbage collection work in Go?"**

Follow-ups:
- What algorithm does Go use? (concurrent tri-color mark and sweep)
- What are the three colors and what do they mean?
- What is `GOGC`? How does it affect GC frequency?
- What is escape analysis? How do you see what escapes to the heap?
- When would a variable escape to the heap vs stay on the stack?
- What is `sync.Pool` and how does it help reduce GC pressure?
- You have a service with high GC pause times. How would you debug and fix this?
- What is the difference between `runtime.GC()` and letting the GC run naturally?

*What they're testing: can you reason about performance and memory allocation in Go.*

---

## Q11: Packages and Modules
**"How do you organize a Go project? What's the difference between a module and a package?"**

Follow-ups:
- What is the `internal/` directory convention? How does Go enforce it?
- When would you use `pkg/` vs `internal/`?
- What is `go.sum`? Why is it committed to version control?
- How does dependency management work? What does `go mod tidy` do?
- How do you handle a diamond dependency problem?
- What is the `init()` function? When does it run? Should you use it?
- How would you structure a monorepo with multiple services in Go?

*What they're testing: project organization maturity and understanding of the module system.*

---

# Round 2 — Concurrency Patterns

## Q12: Worker Pool
**"Design a worker pool in Go. Walk me through the implementation."**

Follow-ups:
- Why limit the number of workers instead of launching a goroutine per job?
- How do you signal workers to stop?
- What happens if a worker panics? How do you handle it?
- How would you implement a worker pool with error collection?
- How do you decide the optimal number of workers?
- What's the difference between a worker pool and `errgroup` with `SetLimit`?
- Your workers talk to an external API with rate limits. How do you adapt?

*What they're testing: can you implement bounded concurrency and handle edge cases.*

---

## Q13: Pipeline
**"Explain the pipeline pattern. When is it useful?"**

Follow-ups:
- How do you handle cancellation in a pipeline? What if an early stage wants to stop?
- What's the relationship between pipeline and fan-out/fan-in?
- How do you handle errors in a pipeline stage? Does the whole pipeline stop?
- What's a goroutine leak in the context of pipelines and how do you prevent it?
- Can you draw the data flow for: generate numbers → filter evens → square → sum?
- How does this relate to Unix pipes?

*What they're testing: understanding of channel composition and resource management.*

---

## Q14: sync.WaitGroup
**"How does WaitGroup work? What are the common mistakes?"**

Follow-ups:
- What happens if you call `wg.Add(1)` inside the goroutine instead of before launching it?
- What happens if `Done()` is called more times than `Add()`?
- Can you reuse a WaitGroup after `Wait()` returns?
- WaitGroup vs channel for waiting on goroutines — when do you use each?
- How would you implement a WaitGroup with a timeout?

*What they're testing: awareness of race conditions in goroutine lifecycle management.*

---

## Q15: Concurrency Design
**"You need to fetch data from 5 different APIs concurrently, with a 3-second overall timeout. Some APIs might fail. How do you design this?"**

Follow-ups:
- How do you collect errors from multiple goroutines?
- What if you only need 3 out of 5 results? (first-N-wins pattern)
- How do you cancel remaining requests when the timeout hits?
- Should you use `errgroup`, channels, or something else? Why?
- What if one API is slow? Should you retry or skip?
- How do you test this? How do you test timeout behavior?

*What they're testing: practical concurrency design, not just knowing the primitives.*

---

# Round 3 — Database & SQL

## Q16: Indexes
**"What is a database index? When would you add one and when would you avoid it?"**

Follow-ups:
- How does a B-tree index work at a high level?
- What is the leftmost prefix rule for composite indexes?
- You have `INDEX(a, b, c)`. Which queries benefit? `WHERE b = ?`? `WHERE a = ? AND c = ?`?
- What is a covering index?
- What's the write penalty of indexes? When does it become a problem?
- Explain the difference between a B-tree index and a hash index.
- When would you use a GIN index?
- How do you identify missing indexes? (slow query log, EXPLAIN)

*What they're testing: can you make indexing decisions, not just define what an index is.*

---

## Q17: Transactions
**"Explain database transactions. What is ACID?"**

Follow-ups:
- Walk me through what happens internally when you `BEGIN`, execute queries, and `COMMIT`.
- What is the WAL (Write-Ahead Log)? Why is it important for durability?
- Explain the difference between Read Committed and Repeatable Read with a concrete scenario.
- Two users transfer money at the same time. Walk me through what could go wrong and how transactions prevent it.
- What is a deadlock? How does PostgreSQL detect and resolve deadlocks?
- What is `SELECT ... FOR UPDATE`? When would you use it?
- What is `FOR UPDATE SKIP LOCKED` and why is it useful for job queues?
- What is an advisory lock? When would you use it instead of row locks?

*What they're testing: do you understand isolation in practice, not just the theory table.*

---

## Q18: Query Optimization
**"This query is slow. How would you debug it?"**

```sql
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.name
HAVING COUNT(o.id) > 5
ORDER BY order_count DESC
LIMIT 10;
```

Follow-ups:
- What does `EXPLAIN ANALYZE` show you? What are you looking for?
- What's the difference between a Seq Scan and an Index Scan? When is Seq Scan actually faster?
- What indexes would help this query?
- Would you index `users.created_at`? What about `orders.user_id`?
- What is a Nested Loop vs Hash Join vs Merge Join? When does the planner choose each?
- The query returns correct results but takes 30 seconds. The users table has 10M rows. What do you do?
- When would you denormalize instead of optimizing the query?

*What they're testing: practical SQL debugging, not just knowing the EXPLAIN keyword.*

---

## Q19: Database Design
**"You're designing the database for an e-commerce platform. Walk me through your schema."**

Follow-ups:
- Users, products, orders, order items — show me the relationships and keys.
- Why foreign keys? What happens on delete?
- How do you handle product prices changing after an order is placed?
- How do you model a product with variants (size, color)?
- At what point would you consider denormalization?
- How do you handle soft deletes vs hard deletes?
- This table will grow to 500M rows. How would you partition it?

*What they're testing: practical schema design, not normalization theory.*

---

## Q20: Migrations
**"How do you manage database schema changes in production?"**

Follow-ups:
- What tools do you use? (goose, golang-migrate, flyway)
- How do you handle a migration that needs to add a NOT NULL column to a table with 100M rows?
- What is a zero-downtime migration? Give an example.
- How do you rollback a failed migration?
- What's the difference between schema migrations and data migrations?
- How do you coordinate migrations in a team? What about concurrent deployments?
- Have you ever had a migration break production? What happened?

*What they're testing: production experience and awareness of real-world complexity.*

---

## Q21: Connection Pooling
**"What is connection pooling and why does it matter?"**

Follow-ups:
- What happens when your pool runs out of connections?
- How do you choose `MaxOpenConns`, `MaxIdleConns`, `ConnMaxLifetime`?
- What happens if you set `MaxOpenConns` too high? Too low?
- Why does `ConnMaxLifetime` exist? What problem does it solve?
- You're seeing "too many connections" errors. What do you check first?
- What's the difference between application-level pooling (database/sql) and an external pooler like PgBouncer?

*What they're testing: production database management experience.*

---

# Round 4 — System Design

## Q22: Design a URL Shortener
**"Design a URL shortening service like bit.ly."**

Follow-ups:
- How do you generate the short code? Base62 encoding? Counter + hash? UUID?
- What are the trade-offs between these approaches?
- How do you handle collisions?
- How do you store the mappings? SQL or KV store?
- What's your redirect strategy — 301 or 302? Why does it matter for analytics?
- How do you handle hot URLs (a viral link getting millions of hits)?
- How would you implement expiring links?
- How would you add analytics (click counts, referrer, geography)?
- This needs to handle 10,000 redirects/second. How do you scale?

---

## Q23: Design a Notification Service
**"Design a service that sends notifications (push, email, SMS)."**

Follow-ups:
- How do you handle different channels (push vs email vs SMS)?
- How do you ensure delivery? What if the push provider is down?
- How do you handle retry? How do you avoid sending duplicate notifications?
- How do you prioritize notifications (transactional vs marketing)?
- How do you support templates? Where do you render them?
- How do you let users configure preferences (which channels, opt-out)?
- The system needs to send 1M emails per hour. How do you architect this?
- How do you track delivery status across all channels?

---

## Q24: Design a Rate Limiter
**"Design a distributed rate limiting system."**

Follow-ups:
- Token bucket vs sliding window vs fixed window — explain each and trade-offs.
- How do you implement this in a distributed system with multiple API servers?
- Where does the state live? Redis? In-memory? Both?
- What HTTP status code and headers should you return?
- How do you handle burst traffic vs sustained traffic?
- How do you rate limit by API key vs by IP vs by user?
- What happens during a Redis failover? Do you allow all traffic or reject all traffic?
- How do you implement tiered rate limits (free tier: 100/min, paid: 1000/min)?

---

## Q25: Design a Service Health Monitoring System
**"Design a system to monitor the uptime of thousands of HTTP endpoints."**

Follow-ups:
- What does "down" mean, exactly? How does your system decide?
- A monitor reports everything is up, but the service's users are complaining. What happened?
- Two of your checker workers disagree on whether an endpoint is currently up. What does your system do?
- How do you know your monitoring system is itself working correctly?
- A customer deletes a monitor with 6 months of check history. Walk me through every implication.

---

## Q26: Design a Chat System
**"Design a real-time chat application."**

Follow-ups:
- WebSockets vs long polling vs SSE — when do you use each?
- How do you handle message ordering?
- How do you store messages? Schema design?
- How do you implement group chat? How does message fan-out work?
- How do you handle a user on multiple devices?
- How do you show "online" status? How does presence work?
- How do you handle message delivery when the user is offline?
- How would you implement end-to-end encryption?

---

## Q27: Scaling Questions (follow any system design)
**"Your service is getting 10x more traffic. What breaks first?"**

Follow-ups:
- Database is the bottleneck. What are your options?
- When do you add a cache? Where in the stack?
- When do you shard? How do you choose a shard key?
- How do you handle cross-shard queries?
- Your read/write ratio is 100:1. How does this affect your architecture?
- How do you handle data consistency across services?
- What's the difference between vertical and horizontal scaling? When do you choose each?
- How do you do zero-downtime deployments?

---

# Round 5 — Architecture & Patterns

## Q28: Clean Architecture
**"How do you structure a Go backend service?"**

Follow-ups:
- What are the layers? What is each layer's responsibility?
- Which direction do dependencies flow? Why?
- Where do you define interfaces — near the implementation or near the consumer?
- How do you handle cross-cutting concerns (logging, tracing)?
- How does your handler layer communicate with the service layer?
- What goes in `cmd/`, `internal/`, `pkg/`?
- How do you handle configuration?
- Show me how you wire dependencies in `main.go`.

*What they're testing: do you structure real projects or just write scripts.*

---

## Q29: Microservices vs Monolith
**"When would you choose microservices over a monolith?"**

Follow-ups:
- What are the actual operational costs of microservices?
- How do you handle transactions across microservices?
- What is the saga pattern? Choreography vs orchestration?
- How do services discover each other?
- How do you handle a breaking change in a shared API?
- You have a monolith that needs to be split. How do you approach it? Where do you start?
- When is a monolith the RIGHT choice? (almost always at the start)
- What is the "distributed monolith" anti-pattern?

*What they're testing: architectural maturity. Junior devs want microservices. Seniors know when not to.*

---

## Q30: API Design
**"How do you design a REST API?"**

Follow-ups:
- What's the difference between PUT and PATCH? When do you use each?
- How do you version your API?
- How do you handle pagination? Offset vs cursor — trade-offs?
- How do you design error responses? What information should they contain?
- How do you handle partial failures in a batch operation?
- REST vs gRPC — when do you use each?
- How do you document your API? (OpenAPI/Swagger)
- How do you handle API authentication? (JWT, API keys, OAuth)
- What is idempotency in API design? How do you implement it for POST requests?

*What they're testing: practical API design sense, not REST theory.*

---

## Q31: Caching
**"When and how do you use caching in a backend service?"**

Follow-ups:
- Cache-aside, write-through, write-behind, read-through — explain each.
- How do you decide what to cache? What should never be cached?
- How do you handle cache invalidation?
- What is cache stampede? How do you prevent it? (singleflight, mutex, probabilistic early expiration)
- What is the thundering herd problem?
- Redis vs Memcached — when do you choose each?
- How do you size your cache? What eviction policy do you use?
- Your cache hit rate is 60%. Is that good or bad? How do you improve it?
- How do you cache in a multi-server environment? What about data consistency?

*What they're testing: caching is easy to add, hard to get right. Do you know the hard parts?*

---

## Q32: Message Queues
**"Explain message-driven architecture. When do you use it?"**

Follow-ups:
- Kafka vs RabbitMQ — fundamental differences and when to choose each.
- What are delivery semantics? At-most-once, at-least-once, exactly-once — explain each.
- How does a Kafka consumer group work? What happens when a consumer dies?
- What is the outbox pattern? What problem does it solve?
- How do you handle message ordering?
- How do you handle poison messages (messages that always fail)?
- What is a dead-letter queue?
- How do you handle schema evolution in messages? (Avro, Protobuf, backward compatibility)
- Synchronous REST call vs async message — how do you decide?

*What they're testing: distributed systems thinking and real messaging experience.*

---

## Q33: Dependency Injection
**"How do you handle dependency injection in Go?"**

Follow-ups:
- Why not just use global variables?
- Manual DI vs wire vs fx — trade-offs?
- How do you handle optional dependencies?
- How does DI help with testing?
- Show me how you'd wire a service with 3 dependencies.
- What is a dependency graph? How do you handle circular dependencies?

*What they're testing: code organization maturity.*

---

# Round 6 — DevOps & Production

## Q34: Docker
**"How do you containerize a Go application?"**

Follow-ups:
- Why multi-stage builds? What's the size difference?
- What is `CGO_ENABLED=0`? Why do you set it?
- How do you handle secrets in Docker? (not in the image!)
- How does Docker networking work? How do containers communicate?
- What is a Docker volume? When do you use it?
- Docker Compose — how do you handle dependency ordering? (`depends_on` is not enough)
- What is the difference between `CMD` and `ENTRYPOINT`?
- How do you debug a container that crashes on startup?

*What they're testing: practical Docker knowledge, not just `docker run`.*

---

## Q35: CI/CD
**"Describe your ideal CI/CD pipeline for a Go service."**

Follow-ups:
- What stages do you include? In what order?
- How do you run integration tests that need a database?
- How do you handle environment-specific configuration?
- What is blue-green deployment? Canary deployment?
- How do you rollback a bad deployment?
- How do you handle database migrations during deployment?
- What tools have you used? (GitHub Actions, GitLab CI, Jenkins, ArgoCD)

*What they're testing: production deployment experience.*

---

## Q36: Observability
**"How do you monitor a Go service in production?"**

Follow-ups:
- What are the three pillars of observability?
- What is structured logging? Why is it better than `fmt.Println`?
- What metrics do you track? (RED method: Rate, Errors, Duration)
- What is distributed tracing? When do you need it?
- How do you correlate logs across microservices? (correlation/request ID)
- What alerts would you set up? How do you avoid alert fatigue?
- You get paged at 3 AM. Walk me through your debugging process.
- What is an SLO? How does it differ from an SLA?

*What they're testing: production mindset. Can you keep systems running?*

---

## Q37: Graceful Shutdown
**"How do you implement graceful shutdown in a Go service?"**

Follow-ups:
- What signals do you catch? (SIGINT, SIGTERM)
- What is the shutdown sequence? Why does order matter?
- How do you drain in-flight HTTP requests?
- What about background workers — how do you stop them cleanly?
- What's your shutdown timeout? What happens when it expires?
- How do you handle a shutdown while a database transaction is in progress?
- How does Kubernetes interact with graceful shutdown? (preStop hook, SIGTERM, grace period)

*What they're testing: production-grade thinking. A toy project crashes on SIGINT. A real one shuts down cleanly.*

---

# Round 7 — Security

## Q38: Authentication
**"How would you implement authentication in a Go API?"**

Follow-ups:
- JWT vs session-based — trade-offs?
- How does JWT work internally? What are the three parts?
- How do you handle JWT expiration? What is a refresh token?
- How do you revoke a JWT? (trick question — JWTs are stateless, so you need a blacklist or short TTL)
- What is OAuth 2.0? Explain the Authorization Code flow.
- What is the difference between authentication and authorization?
- How do you store passwords? (bcrypt, argon2 — never MD5/SHA)
- What is CORS? When do you need it?

*What they're testing: security fundamentals that every backend developer must know.*

---

## Q39: Common Vulnerabilities
**"What security concerns do you think about when building a backend API?"**

Follow-ups:
- What is SQL injection? How does Go's `database/sql` prevent it?
- What is XSS? Is it relevant for a backend API? (yes, if you render HTML or return user input)
- What is CSRF? How do you prevent it?
- How do you handle sensitive data in logs?
- How do you manage secrets (DB passwords, API keys)?
- What is rate limiting and why is it a security concern?
- What is the principle of least privilege? How do you apply it?

*What they're testing: security awareness. You don't need to be a security expert, but you need to not write vulnerable code.*

---

# Round 8 — Behavioral & Situational

## Q40: About You
**"Tell me about yourself."** (2 minutes max)

**"Why are you looking for a new role?"**

**"Why Go? Why backend?"**

**"What have you been doing the last 4 months?"**

*See [[Go Backend Interview Preparation]] Part 9 for prepared answers.*

---

## Q41: Technical Challenges
**"Tell me about the most challenging technical problem you've solved."**

Follow-ups:
- What was the impact of the problem?
- What alternatives did you consider?
- How did you know your solution worked?
- What would you do differently in hindsight?

Use STAR: **Situation** → **Task** → **Action** → **Result**

Pick from:
- Scam detection system with byte matching
- Payment processing / bank configs migration
- A game architecture challenge (state machines, ECS, rendering pipeline)
- Performance optimization (any project)

---

## Q42: Collaboration
**"Tell me about a time you disagreed with a teammate on a technical decision."**

Follow-ups:
- How did you present your case?
- What did you learn from their perspective?
- What was the outcome?
- Would you do anything differently?

---

## Q43: Failure
**"Tell me about a time you made a mistake in production."**

Follow-ups:
- How did you discover it?
- What was the blast radius?
- How did you fix it?
- What did you change to prevent it from happening again?

---

## Q44: Working Style
**"How do you approach a new codebase?"**

**"How do you handle tight deadlines with competing priorities?"**

**"How do you decide when something is 'good enough' to ship?"**

**"What does a good code review look like?"**

---

## Q45: Questions YOU Ask (ask at least 3)

These signal seniority and genuine interest:

- "What does the architecture look like? How many services, what patterns?"
- "How do you handle deployments? How often do you deploy to production?"
- "What's the biggest technical challenge the team is dealing with right now?"
- "How does the on-call rotation work?"
- "What does the development workflow look like? PR size, review process?"
- "How do you handle technical debt? Is there time allocated for it?"
- "What does success look like in the first 90 days?"
- "What's the team culture around testing?"
- "How do you make architectural decisions? Who's involved?"

Never ask:
- Things easily found on the website
- Salary in a technical interview round
- "Do I have the job?"

---

# How to Use This Document

## Week 1 (while building the project)
Each evening, pick 3-5 questions from the relevant round. Answer them out loud without looking at the answers. Record which ones tripped you up → those are your study targets for the next day.

## Week 2 (mock interviews)
Go through entire rounds end-to-end:
- Set a timer (45 minutes)
- Pick a round
- Answer every question out loud as if an interviewer is sitting across from you
- Score yourself honestly: Could I answer the follow-ups? Did I give concrete examples?

## The Day Before
Pick one question from each round. Answer it. If you can handle the follow-ups, you're ready.

---

# Scoring Yourself

For each question, rate yourself:

| Score | Meaning |
|-------|---------|
| 1 | Can't answer the initial question |
| 2 | Can answer the initial question but not follow-ups |
| 3 | Can answer most follow-ups but lack concrete examples |
| 4 | Can answer all follow-ups with real examples and trade-offs |
| 5 | Can teach this topic to someone else and answer questions I haven't seen |

**Target: 3+ on everything, 4+ on Go and Database rounds.**

Anything scored 1-2 → study that topic deeply using [[Go Backend Deep Learning Method]]
Anything scored 3 → review in [[Go Backend Interview Preparation]] and practice explaining out loud
Anything scored 4-5 → you're good, move on

---

# Round 9 — Observability Domain

> This round is for companies where observability is the product: Grafana Labs, Datadog, Chronosphere, Honeycomb, VictoriaMetrics. These companies hire backend engineers to build the systems every other engineering team depends on to keep their services alive.
>
> Rounds 1–8 are still the foundation. Observability companies will ask Go, database, and system design questions. This round is the domain layer on top. Start here only after you've instrumented your job processing service — you need that concrete experience before these questions are meaningful.

---

## Q46: What Is a Metric
**"What is a metric? How is it different from a log?"**

Follow-ups:
- What is a time-series? Define it precisely.
- What are labels (also called dimensions or tags)? Give an example.
- What is cardinality? What makes cardinality high or low?
- What is a cardinality explosion? Give a concrete example of how it happens.
- Why does high cardinality destroy a time-series database?
- You have a metric `http_requests_total` with labels `method`, `status`, `path`. The `path` label contains full URL paths including UUIDs. What goes wrong?
- How would you fix that cardinality problem?
- What is the difference between a counter, gauge, and histogram?

*What they're testing: whether you understand the fundamental data model they work with every day, and whether you can reason about the cardinality problem that kills real production systems.*

---

## Q47: Prometheus Data Model and Scrape Model
**"How does Prometheus work? Walk me through how it collects data."**

Follow-ups:
- What is the Prometheus data model? What uniquely identifies a time series?
- What is the scrape model? How is it different from a push model?
- What are the trade-offs between pull (scrape) and push? When would you prefer push?
- What is a Prometheus exporter? Give three examples.
- What is a target? How does Prometheus discover targets?
- What is a stale marker? What happens when a target disappears?
- When would you use Prometheus Pushgateway, and what are its limitations?
- You have a batch job that runs for 30 seconds and exits. How do you get its metrics into Prometheus?

*What they're testing: do you understand the architecture, not just the query language. The pull vs push trade-off comes up constantly at Prometheus-adjacent companies.*

---

## Q48: Prometheus TSDB Internals
**"How does Prometheus store data internally?"**

Follow-ups:
- What is a block? How long does a block cover by default?
- What is the WAL (Write-Ahead Log)? Why does Prometheus need one?
- What is the head block? How does it differ from persisted blocks?
- What is compaction? Why does it happen? What does it produce?
- What is XOR encoding and why is it efficient for time-series data?
- What happens to data during a crash before it is written to a block?
- What is out-of-order ingestion? How does modern Prometheus handle it?
- You have 2 million active series. Prometheus is consuming 16 GB of RAM. What determines memory usage and how would you reduce it?

*What they're testing: Grafana Labs engineers built Thanos and Mimir on top of Prometheus TSDB — they expect you to understand what happens under the hood, not just write PromQL.*

---

## Q49: PromQL
**"Write me a PromQL query for the per-second request rate over the last 5 minutes."**

Follow-ups:
- What is the difference between `rate()` and `irate()`? When do you use each?
- What is a range vector vs an instant vector?
- What does `increase()` do? How is it different from `rate()`?
- What is the `by` clause? What is `without`?
- Write a query for the 99th percentile latency of `http_request_duration_seconds` (a histogram).
- Why can't you take the average of percentiles across instances?
- What is `histogram_quantile`? What are its accuracy limitations?
- What is a recording rule? Why would you create one?
- You have a query that takes 45 seconds to evaluate. What would cause that and how would you fix it?

*What they're testing: practical PromQL fluency. The histogram_quantile averaging trap and recording rules are real questions from Grafana Labs interviews.*

---

## Q50: OpenTelemetry
**"What is OpenTelemetry? Why does it exist?"**

Follow-ups:
- What problem was OpenTelemetry created to solve? (vendor lock-in, fragmentation)
- What are the three signals? Define each in one sentence.
- What is the difference between the OTel SDK and the OTel Collector?
- What is an exporter? Give examples.
- What is OTLP? Why was a new protocol needed?
- What is automatic instrumentation vs manual instrumentation?
- Your team uses Prometheus today and wants to migrate to OpenTelemetry for metrics. What changes?
- What is a Resource in OpenTelemetry? What attributes does it typically carry?

*What they're testing: awareness of the ecosystem and standardization push. Every observability company either integrates with OTel or competes with it.*

---

## Q51: Distributed Tracing
**"Explain distributed tracing. What problem does it solve that logs don't?"**

Follow-ups:
- What is a span? What information does a span carry?
- What is a trace? How are spans connected into a tree?
- What is trace context propagation? How does it work across HTTP calls?
- What is the W3C TraceContext standard? What are `traceparent` and `tracestate` headers?
- What is the difference between head-based sampling and tail-based sampling?
- What are the trade-offs of head-based sampling? When does it fail you?
- Why is tail-based sampling harder to implement?
- What is Jaeger? What is Tempo? How are they different architecturally?
- You have a trace showing a 2-second gap between two spans. How do you interpret that?

*What they're testing: can you reason about distributed systems through traces. Head vs tail sampling is a real design question at every company running a tracing backend.*

---

## Q52: Log Aggregation
**"How does log aggregation work? Walk me through how a log gets from an application to a dashboard."**

Follow-ups:
- What is structured logging? Why is it better than unstructured log lines?
- What is the difference between indexing logs and not indexing them?
- Trade-offs of full-text indexing (Elasticsearch) vs label-based indexing (Loki)?
- What is Loki's data model? How does it differ from Elasticsearch?
- Why is Loki cheaper to operate than Elasticsearch at scale?
- What is log parsing? Why is parsing expensive?
- You have a service logging 100,000 lines/minute. What are the operational concerns?
- When is it appropriate to drop logs (sampling)?
- How do you correlate a log entry with a trace? What fields do you need?

*What they're testing: Loki is a Grafana Labs product. They want to know if you understand why their architectural choices (no full-text index) are trade-offs, not deficiencies.*

---

## Q53: Alerting and SLOs
**"What is an SLO? How is it different from an SLA?"**

Follow-ups:
- Define SLO, SLA, and SLI precisely.
- What is an error budget? How do you calculate it?
- Your SLO is 99.9% availability over 30 days. You've used 80% of your error budget in the first 10 days. What do you do?
- What is a burn rate alert? Why is it better than a simple threshold alert?
- What is a 1-hour burn rate vs a 6-hour burn rate alert? Why do you need both?
- What makes a good alert? What makes an alert bad?
- What is alert fatigue? How does it happen and how do you fix it?
- You're setting up a latency alert. The p50 is fine but p99 is high. Which do you alert on?
- What is the difference between alerting on symptoms vs causes?

*What they're testing: SLO-based alerting is the foundation of products like Grafana SLO, Datadog SLOs, and Chronosphere. Understanding it shows you know the domain, not just the tooling.*

---

## Q54: System Design — Metrics Ingestion at Scale
**"Design a system that ingests 1 million time-series data points per second."**

Follow-ups:
- Walk me through every component a data point touches from agent to storage.
- How do you handle fan-out from thousands of agents writing to one backend?
- What is `remote_write` in Prometheus? What are its back-pressure semantics?
- How do you shard writes? What is the shard key?
- How do you handle out-of-order ingestion?
- Where is the first durability checkpoint in your design?
- How do you handle a 10x traffic spike? What degrades gracefully, what falls over?
- What is the core idea behind Thanos/Cortex/Mimir? How do they solve the Prometheus scalability problem?
- Which components are stateful vs stateless? What does that mean for scaling?

*What they're testing: this is the core system design question at Grafana Labs, VictoriaMetrics, and Chronosphere. They want to see write path, durability, sharding, and back-pressure reasoning — not "add more servers."*

---

## Q55: System Design — Alerting Engine
**"Design an alerting engine. Given alert rules, evaluate them continuously and fire notifications."**

Follow-ups:
- How do you schedule rule evaluation? How do you avoid thundering herd (all rules at once)?
- What are the trade-offs of shorter vs longer evaluation intervals?
- How do you handle the metrics backend being slow or unavailable during evaluation?
- What states can an alert be in? (inactive, pending, firing)
- What is a `for` duration in an alert rule? Why does it exist?
- How do you prevent alert flapping?
- How do you deliver notifications reliably if the target (PagerDuty, Slack) is down?
- How do you deduplicate alerts across multiple evaluator instances?
- What does Alertmanager do that the rule evaluator does not?

*What they're testing: alerting is a stateful distributed system problem, not just query evaluation. Grafana Mimir and Thanos have ruler components — this question reveals whether you've thought about the hard cases.*

---

## Q56: Go Instrumentation — prometheus/client_golang
**"How do you instrument a Go service with Prometheus metrics?"**

Follow-ups:
- What are the four metric types? What is each one for?
- Show me how you register and increment a Counter with labels.
- What is the danger of passing arbitrary strings as label values?
- What is a Histogram? What are buckets? How do you choose bucket boundaries?
- What is the difference between a Histogram and a Summary? Why prefer Histograms?
- How do you expose metrics over HTTP?
- You have a Histogram tracking request latency. The p99 from `histogram_quantile` looks wrong. What could cause that?
- What is `promauto`? When would you use it instead of manual registration?

*What they're testing: hands-on Go instrumentation. At observability companies this is table stakes — you instrument your own services and sometimes work on the client libraries themselves.*

---

## Q57: Go Instrumentation — OpenTelemetry Go SDK
**"How do you add tracing to a Go service using the OpenTelemetry Go SDK?"**

Follow-ups:
- What is a `TracerProvider`? Why configure it globally vs passing it around?
- What is a `Tracer`? How do you get one?
- Show me how you start and end a span. What is the Go idiom for span cleanup?
- How do you add attributes and record errors on a span?
- How does context propagation work in Go? How does a child span find its parent?
- What is the `otelhttp` package? What does it do automatically?
- How do you configure an exporter to send traces to Tempo or Jaeger?
- What is a `Propagator`? Why might you need to configure it explicitly?

*What they're testing: practical OTel Go experience. Context propagation is the part candidates most often get wrong.*

---

## Q58: Long-Term Storage and Downsampling
**"Prometheus only retains 15 days by default. How do you store metrics long-term?"**

Follow-ups:
- What is the challenge with querying months of raw 15-second interval data?
- What is downsampling? What resolutions do systems like Thanos typically keep?
- What data do you lose when you downsample? Why does that matter for alerting?
- How does Thanos store long-term data? What is an object store and why is it appropriate here?
- What is a Thanos Compactor? What does it do?
- What is the query path when a user asks for 1 year of data?
- How do you implement retention policies? Who is responsible for deleting old data?
- Trade-off: storing raw data vs pre-aggregated data?
- You need to store 18 months of metrics for 500,000 series. Estimate storage cost and walk me through the architecture.

*What they're testing: long-term storage is the unsolved problem in vanilla Prometheus. Thanos, Cortex, Mimir, and VictoriaMetrics exist to solve it. Understanding the trade-offs shows you know the domain, not just the tools.*
