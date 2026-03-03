# Go Backend — Deep Learning Method

> Build the project below. When you finish a piece and want to check your understanding, find the relevant section and answer the questions.
>
> If the answers hold up, move on. If they fall apart, you found a gap — go fill it.
>
> Not a schedule. Not steps to follow in order.

See [[Go Backend Interview Preparation]] when you blank on syntax. See [[Go Backend Interview Questions Bank]] to stress-test against real interview questions.

---

# The Project — Service Health Monitor

You're building a tool that checks whether HTTP endpoints are up or down, detects outages, and provides a status page. Think BetterStack or Pingdom, but yours.

**Why this project:** It's a real product category. It covers exactly the technical problems Go backend interviews probe — concurrency, state machines, time-series data, caching, distributed coordination. On Day 8, you instrument a monitoring tool with monitoring tools, which is the one story that lands at any observability company.

## What it does

**Monitors**: An HTTP endpoint you want to watch. Has a URL, check interval, timeout, and failure threshold (consecutive failures before opening an incident).

**Checks**: Every interval, a worker hits the URL and records the result — status code, latency, success or failure.

**Incidents**: When a monitor crosses its failure threshold, an incident opens. When checks recover, it closes. You decide the recovery logic.

**Status page**: A public endpoint showing current status for all monitors.

## API

```
POST   /api/v1/monitors              create monitor
GET    /api/v1/monitors              list monitors
GET    /api/v1/monitors/{id}         get monitor
DELETE /api/v1/monitors/{id}         delete monitor
GET    /api/v1/monitors/{id}/checks  check history
GET    /api/v1/status                public status page (cached)
GET    /metrics                      Prometheus metrics (see observability section)
```

## CLI

```
healthmon monitor add --name "API" --url "https://api.example.com" --interval 60
healthmon monitor list
healthmon monitor delete <id>
healthmon status
```

## Schema (starting point — your decisions may differ)

```sql
CREATE TABLE monitors (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name         TEXT NOT NULL,
    url          TEXT NOT NULL,
    interval_sec INT  NOT NULL DEFAULT 60,
    timeout_sec  INT  NOT NULL DEFAULT 10,
    threshold    INT  NOT NULL DEFAULT 3,
    status       TEXT NOT NULL DEFAULT 'pending',
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE checks (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    monitor_id  UUID NOT NULL REFERENCES monitors(id) ON DELETE CASCADE,
    status      TEXT NOT NULL,
    latency_ms  INT,
    status_code INT,
    error       TEXT,
    checked_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE incidents (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    monitor_id  UUID NOT NULL REFERENCES monitors(id) ON DELETE CASCADE,
    message     TEXT,
    status      TEXT NOT NULL DEFAULT 'open',
    started_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    resolved_at TIMESTAMPTZ
);
```

## Folder structure

```
healthmon/
├── cmd/
│   ├── api/main.go
│   └── cli/main.go
├── internal/
│   ├── domain/monitor.go       Monitor, Check, Incident types
│   ├── service/monitor.go      business logic
│   ├── repository/postgres.go  DB queries
│   ├── handler/http.go         HTTP handlers
│   └── checker/checker.go      concurrent HTTP checkers
├── docker-compose.yml
└── go.mod
```

---

# Stress-Test Questions

Use these when you've built a piece and want to check your understanding. Answer them before moving on. If something doesn't hold, that's where to go deeper.

## After you've set up structure and domain types

- What does your domain layer know about? What is it explicitly ignorant of?
- Which fields on Monitor constrain behavior, and which are just data?
- You put things in certain packages. What drove those boundaries?

## After you've built the schema and repositories

- What operations does your schema make fast? What does it make slow or impossible?
- `ON DELETE CASCADE` on checks and incidents — what does it give you? What does it take away?
- The checks table grows with every check, forever. What are your options, and when does each make sense?
- `checked_at` — indexed or not? How would you decide?

## After you've built the checker

- You have N monitors and N goroutines. What happens as N grows? When does it become a problem?
- Two goroutines happen to check the same URL at the same time — problem, neutral, or useful?
- A check hangs. The target accepts the TCP connection but never responds. Walk through exactly what happens in your code right now.
- Check timeout vs `http.Client` timeout — are these the same thing?

## After you've built incident detection

- Draw the complete state machine for a monitor's status. Are there transitions you haven't handled?
- Monitor fails once, recovers, fails again, recovers, then fails three more times. Trace through your logic. When does an incident open?
- "Recovered" — what does that mean exactly in your implementation? What are the implications of each possible definition?
- Two checker goroutines can update the same monitor state simultaneously. What does your code do right now?

## After you've added caching

- Why this endpoint and not the others?
- A monitor goes down. How long before the status page reflects it? What controls that delay?
- Redis goes down. What does your application do right now — is that what you want?
- What did you give up by adding the cache?

## After you've written tests

- Which of your tests would catch a bug you might plausibly write?
- Which of your tests only confirm that the code ran?
- Your checker test uses a fake HTTP server. What categories of bugs would that miss entirely?
- What's the hardest part of your system to test? Why?

## After you've containerized it

- What's the startup dependency order? What breaks if it's violated?
- What's inside your Docker image? What would someone expect to be there that isn't?
- A container reports itself as healthy but the service is broken. What could that look like?
- What does the multi-stage build buy you? What does it cost?

## After you've added CLI and observability

- Every label you attached to your metrics — what makes a label dimension useful? Where does that reasoning break down?
- You're observing your own system from the outside. What can't you see?
- Your dashboard shows all monitors are up. A user tells you the status page is wrong. Which metric would you look at — do you have it?
- You want to add a label for the specific monitor URL. Before you do: what does that mean for the total number of unique metric series Prometheus tracks?

See [[Go Backend Observability Track]] for the instrumentation code and domain knowledge.

## When the whole thing is built — system design

Answer these in full. Not bullet points — explanations with trade-offs. Then go back and find where you waved your hands.

- Your system works at 50 monitors. What starts to break first as that number grows?
- What is the relationship between check interval and total system load?
- Your status page shows a monitor as "up." The monitor's actual users are reporting errors. What happened?
- Describe every trade-off you made while building this. What was on each side?
- What does "uptime percentage" mean in your system? What's imprecise about it?

---

# Contrast and Compare

Not about the project. Pure reasoning — pairs of concepts where the interesting content is the comparison, not the definition.

For each pair: define both sides without looking anything up, describe when you'd choose one over the other, find a case where the choice is genuinely hard, identify something the two have in common that isn't obvious.

**Concurrency**:
- Goroutines vs OS threads
- Channels vs mutexes for coordinating state
- `errgroup` vs manual goroutine management with `WaitGroup`
- `context.WithCancel` vs `context.WithTimeout`

**Data**:
- SQL transaction vs application-level coordination
- Read replica vs cache
- Index scan vs sequential scan
- Optimistic locking vs pessimistic locking

**Architecture**:
- Synchronous call vs event publishing
- Pull model vs push model
- Monolith vs service split at a specific seam
- Configuration at build time vs runtime

**Go-specific**:
- Interface vs concrete type in a function signature
- Pointer receiver vs value receiver
- `make(chan T, N)` vs `make(chan T, 0)`
- Embedding vs composition

---

# One Final Thing

The interview is not a knowledge quiz. It's a conversation about how you think.

An interviewer doesn't ask about index design to hear the textbook answer. They're checking whether you've ever had a real reason to think about it — because real understanding only comes from real stakes.

Your project gives you that reason. When index design comes up, you have a specific problem you actually faced, with specific trade-offs you actually weighed. That can't be faked under follow-up questions.

The documents don't create that. The building does.
