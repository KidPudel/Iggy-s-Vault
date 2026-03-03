# Go Backend — Observability Engineering Track

> This is Phase 2. Complete [[Go Backend Deep Learning Method]] first — specifically Day 8.
>
> Foundation: [[Go Backend Interview Preparation]] · [[Go Backend Interview Questions Bank]]

---

# The Honest Picture

Grafana Labs, Datadog, Honeycomb, Chronosphere, VictoriaMetrics — these companies build the tools that other engineers use to understand their systems. You'd be a product engineer building the actual product, not just using it.

**What they actually want:**
- Strong Go (you're building this)
- Distributed systems thinking (your foundation builds this)
- Genuine curiosity about the domain (you develop this deliberately)
- Signal that you've engaged with observability at a technical level (Day 8 does this)

**What they don't require on day one:**
- Deep knowledge of every TSDB implementation detail
- Prior experience building observability tooling at scale
- Years of SRE/ops work

**The gap between "never thought about how metrics work" and "interview-ready for Grafana Labs" is roughly 8–10 weeks of focused work from where you are.** That means: get a job in 2 weeks, work, go deep in parallel, apply from a position of strength.

---

# The Bridge (Day 8 of Phase 1)

Day 8 of the Deep Learning Method is: instrument your health monitor with Prometheus and OpenTelemetry.

This is what turns you from "backend engineer applying to Grafana Labs" to "engineer who has worked with the exact tools Grafana Labs builds." Do it before reading the rest of this document.

**Your story for observability interviews comes from Day 8's Interrogate questions, not from this document.** Answer those questions, build the instrumentation, and then you'll have something real to talk about.

## What to add

### 1. Prometheus metrics

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    checksTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "checks_total",
            Help: "Total checks performed, by result status",
        },
        []string{"monitor", "status"}, // status: up, down, timeout
    )

    checkLatency = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "check_latency_seconds",
            Help:    "HTTP check latency in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"monitor"},
    )

    monitorsUp = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "monitors_up",
        Help: "Number of monitors currently reporting UP",
    })

    openIncidents = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "open_incidents",
        Help: "Number of currently open incidents",
    })
)

func init() {
    prometheus.MustRegister(checksTotal, checkLatency, monitorsUp, openIncidents)
}

// In your checker:
func (c *Checker) runCheck(ctx context.Context, monitor *domain.Monitor) {
    start := time.Now()
    result, err := c.httpCheck(ctx, monitor)
    duration := time.Since(start).Seconds()

    checkLatency.WithLabelValues(monitor.Name).Observe(duration)

    status := "up"
    if err != nil || result.StatusCode >= 500 {
        status = "down"
    }
    checksTotal.WithLabelValues(monitor.Name, status).Inc()
}

// Expose the metrics endpoint:
mux.Handle("GET /metrics", promhttp.Handler())
```

### 2. OpenTelemetry tracing

```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/codes"
    "go.opentelemetry.io/otel/trace"
    otlptracegrpc "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.21.0"
)

// Setup in main.go:
func setupTelemetry(ctx context.Context) (func(context.Context) error, error) {
    exporter, err := otlptracegrpc.New(ctx,
        otlptracegrpc.WithEndpoint("localhost:4317"),
        otlptracegrpc.WithInsecure(),
    )
    if err != nil {
        return nil, err
    }

    tp := sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),
        sdktrace.WithSampler(sdktrace.AlwaysSample()),
        sdktrace.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceNameKey.String("healthmon"),
        )),
    )

    otel.SetTracerProvider(tp)
    otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
        propagation.TraceContext{},
        propagation.Baggage{},
    ))

    return tp.Shutdown, nil
}

// In a handler:
func (h *MonitorHandler) CreateMonitor(w http.ResponseWriter, r *http.Request) {
    ctx, span := otel.Tracer("healthmon").Start(r.Context(), "handler.CreateMonitor")
    defer span.End()

    monitor, err := h.service.CreateMonitor(ctx, req)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return
    }
    span.SetAttributes(attribute.String("monitor.id", monitor.ID))
}

// In the checker:
func (c *Checker) runCheck(ctx context.Context, monitor *domain.Monitor) error {
    ctx, span := otel.Tracer("healthmon").Start(ctx, "checker.runCheck",
        trace.WithAttributes(
            attribute.String("monitor.id", monitor.ID),
            attribute.String("monitor.url", monitor.URL),
        ),
    )
    defer span.End()
    // ...
}
```

### 3. Structured logging with trace correlation

```go
import "log/slog"

logger := slog.With(
    "monitor_id", monitor.ID,
    "monitor_url", monitor.URL,
    "trace_id", span.SpanContext().TraceID().String(),
    "worker_id", c.ID,
)

logger.Info("check completed", "status", result.Status, "latency_ms", result.LatencyMs)
logger.Error("check failed", "error", err, "consecutive_failures", monitor.ConsecutiveFailures)
```

### 4. docker-compose additions

```yaml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
  ports: ["9090:9090"]

grafana:
  image: grafana/grafana:latest
  ports: ["3000:3000"]
  environment:
    - GF_AUTH_ANONYMOUS_ENABLED=true
```

`prometheus.yml`:
```yaml
scrape_configs:
  - job_name: 'healthmon'
    static_configs:
      - targets: ['api:8080']
```

Build a dashboard showing at minimum: check rate by status, check latency percentiles (p50/p95/p99), monitors currently up/down.

---

# Domain Knowledge Map

Work through these in order. Level 1 before Level 2, etc.

## Level 1 — Concepts

### The three signals

**Metrics**: Numeric measurements over time. Aggregatable. Low cardinality. Answer "how many" and "how fast."
- Counter: only goes up (request count, error count)
- Gauge: can go up or down (queue depth, active connections)
- Histogram: distribution of values (request duration) — gives you percentiles

**Logs**: Discrete events with context. High cardinality. Answer "what happened."
- Structured logs (JSON) are queryable; unstructured logs (printf strings) are not
- Correlation ID links logs across a request's journey

**Traces**: Causal chain of operations for a single request.
- Span: a named, timed operation with attributes
- Parent-child span relationships
- Context propagation carries the trace ID across service boundaries (HTTP headers, gRPC metadata)

### Time-series data

A time-series is a sequence of (timestamp, value) pairs for a unique combination of labels.

```
# Prometheus text format
http_requests_total{method="GET", status="200", handler="/monitors"} 1234 1700000000000
http_requests_total{method="POST", status="500", handler="/monitors"} 5 1700000000000
```

Each unique `{metric_name + label set}` is a **series**.

### Cardinality

The number of unique series. This is the central scaling problem in metrics.

```
# Low cardinality: 3 methods × 4 statuses × 10 handlers = 120 series. Fine.
http_requests_total{method, status, handler}

# High cardinality: user_id has millions of values → millions of series. Kills Prometheus.
http_requests_total{method, status, handler, user_id}
```

**Cardinality explosion**: when a label has unbounded values (user IDs, trace IDs, UUIDs). Understanding this is what separates someone who has built with metrics from someone who has only read about them.

**Rule**: labels describe dimensions you want to query on (group by, filter), not individual instances.

### Sampling (for traces)

You cannot store every trace at scale.

- **Head sampling**: decide at the start of a request — cheap, but you lose rare/interesting traces
- **Tail sampling**: buffer all spans for a request, decide after it completes — captures errors and slow requests, but requires buffering infrastructure
- **Adaptive sampling**: adjust rate based on traffic patterns

---

## Level 2 — Tools (learn by using)

### Prometheus

- **Scrape model**: Prometheus pulls metrics from targets on a schedule. Pull, not push.
- **TSDB**: embedded time-series database
  - Data organized into **blocks** (2-hour windows by default)
  - Each block: chunks (compressed data) + index + meta.json
  - **WAL** (Write-Ahead Log): recent data lives here before compaction
  - **Head block**: in-memory block for the most recent data
  - **Compaction**: blocks merge over time — reduces duplication, improves query speed
- **Retention**: default 15 days. Old blocks deleted.

**PromQL** — enough to read and write queries:

```promql
# Rate of requests per second over 5m window
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# p95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Per-instance
sum by (instance) (rate(http_requests_total[5m]))
```

Key concepts: instant vectors vs range vectors, `rate()` vs `irate()`, `histogram_quantile()`, label matchers (`=`, `!=`, `=~`, `!~`), `sum by`, `avg by`.

### OpenTelemetry

The standard for instrumenting software. Replaces proprietary agents with a single SDK.

```
Your Service → OTel SDK → OTel Collector → Backend (Jaeger/Tempo/Datadog)
```

- **SDK**: in-process library you call to create spans, record metrics, emit logs
- **Collector**: separate process that receives telemetry, batches and processes it, exports to backends. Decouples your service from the backend.
- **OTLP**: the wire protocol (gRPC or HTTP/JSON)
- **Semantic conventions**: standard attribute names (`http.method`, `db.system`) so backends can process consistently

### Grafana Loki

Log aggregation designed to work like Prometheus. Indexes only labels, not log contents. Much cheaper storage than Elasticsearch. Trade-off: full-text search is slower.

```logql
# Find error logs from the checker
{app="healthmon", component="checker"} |= "error"

# Parse and filter structured logs
{app="healthmon"} | json | latency_ms > 1000
```

### Grafana Tempo

Distributed tracing backend. Stores traces in object storage (S3/GCS). Queries via TraceQL. Tight integration with Grafana.

---

## Level 3 — System Design (the hard problems)

These are what observability companies actually work on. You don't need to solve them — understand them well enough to discuss intelligently.

### Ingestion at scale

**Problem**: Ingest millions of metric samples per second from thousands of targets.

**Architecture pattern** (Cortex/Mimir/Thanos style):
```
Prometheus instances → Distributor → Ingesters (ring) → Object storage
                                          ↓
                                    Compactor (background)
                                          ↓
                              Querier (reads ingester + storage at query time)
```

- **Consistent hashing ring**: distributes series across ingesters by hash of series labels
- **Replication factor**: write to N ingesters for durability (typically 3)
- **Ingester**: holds recent data in memory + WAL, flushes to long-term storage periodically

Key problems: handling ingester failure (re-sharding), out-of-order samples, memory pressure during cardinality spikes.

### Long-term storage

Prometheus local TSDB is not designed for years of retention.

- **Thanos**: adds object storage to Prometheus. Sidecar uploads blocks as created.
- **Cortex/Mimir**: horizontally scalable Prometheus-compatible backend. Multi-tenant.
- **VictoriaMetrics**: alternative TSDB. More storage-efficient, faster queries.

### Alerting at scale

Simple alerting: evaluate PromQL query every 30s → if result matches condition → fire alert.

Hard problems:
- **Deduplication**: same condition firing on multiple Prometheus instances
- **Grouping**: don't fire 1000 alerts, fire one with context
- **Routing**: right alert to right team
- **Inhibition**: suppress lower-priority alerts when a higher-priority one fires

Alertmanager handles this for the Prometheus ecosystem.

### SLOs and error budgets

- **SLI** (Service Level Indicator): the metric you're measuring (request success rate)
- **SLO** (Service Level Objective): the target (99.9% success over 30 days)
- **Error budget**: the allowed failure (0.1% of requests = ~43 minutes/month)
- **Alert on burn rate**: alert when consuming error budget too fast, not when you hit the limit

---

## Level 4 — Go instrumentation (reference)

Libraries for your health monitor service:

```go
// Prometheus
import "github.com/prometheus/client_golang/prometheus"
import "github.com/prometheus/client_golang/prometheus/promauto"
import "github.com/prometheus/client_golang/prometheus/promhttp"

// OpenTelemetry
import "go.opentelemetry.io/otel"
import "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
import sdktrace "go.opentelemetry.io/otel/sdk/trace"
import "go.opentelemetry.io/otel/propagation"

// Logging (structured)
import "log/slog" // stdlib, Go 1.21+
```

---

# Phase 2 Project

Once Phase 1 is done (health monitor with observability instrumentation), build one of these.

## Option A: Simple Metrics Collector (recommended for Grafana Labs target)

A service that:
1. Accepts metric samples via HTTP (Prometheus remote write protocol or simple JSON)
2. Stores them in a time-series structure (memory + WAL)
3. Exposes a query API (range query: samples for `{metric, labels}` between T1 and T2)
4. Has configurable retention

Forces you to design a time-series data structure, implement the ingestion path, handle retention, and understand query performance. You'll understand why TSDB is hard.

## Option B: Log Aggregation Service

A service that:
1. Accepts structured logs via HTTP (Loki push API format)
2. Stores logs indexed by labels only (not full-text)
3. Allows querying by label filters and time range

Forces you to understand label indexing, the trade-off between index size and query speed, and storage layout for append-heavy workloads.

## Option C: Alerting Engine

A service that:
1. Accepts alert rules (metric name + threshold + duration)
2. Periodically evaluates rules against a metrics source
3. Fires alerts (webhook POST) when rules are violated
4. De-duplicates and groups alerts

Forces you to understand the alerting pipeline, state machines, and evaluation at scale.

---

# Realistic Timeline

| Phase | Duration | Goal |
|-------|----------|------|
| **Foundation** (current) | 2 weeks | Any Go backend job |
| **Bridge** (Day 8 of Phase 1) | part of the 2 weeks | Instrument health monitor → concrete domain story |
| **Domain knowledge** | 2–3 weeks | Work through Levels 1–3 of this document + engineering blogs |
| **Phase 2 project** | 2–3 weeks | Build one of the options above |
| **Target applications** | ongoing | Grafana Labs, Datadog, VictoriaMetrics, Honeycomb, SigNoz |

**Total to be competitive at observability companies: ~8–10 weeks from now.**

---

# Engineering Blogs (Primary Research)

Read these after you've built. The problems will make sense once you've had to think about them yourself.

- **Grafana Labs**: grafana.com/blog/tags/engineering — Loki, Mimir, Tempo architecture posts
- **Thanos**: thanos.io/blog — distributed Prometheus at scale
- **Datadog**: datadoghq.com/blog/engineering — high-scale metrics ingestion
- **Honeycomb**: honeycomb.io/blog — observability philosophy, wide events vs metrics
- **VictoriaMetrics**: victoriametrics.com/blog — storage engine optimization, benchmarks

Focus on: storage design, cardinality problems, ingestion pipelines, query optimization for time-series data.

---

# Security Domain (secondary)

If you're also interested in security companies (Snyk, Wiz, Aqua Security), the overlap with observability is meaningful: both involve high-throughput data ingestion, event stream processing, and policy evaluation.

**Domain knowledge to add later:**

**Container/OCI internals**: An OCI image is a stack of layers. Each layer is a tar.gz archive with file changes. The image manifest references layer digests. Understanding this is essential for container scanning products.

**Vulnerability databases**: CVE/NVD, GHSA (GitHub Advisory), OSV. Structured data about known vulnerabilities mapped to package versions. Security scanners cross-reference your dependency tree against these databases.

**Dependency trees**: npm, pip, go modules — each has a different resolution algorithm. Transitive dependencies (A requires B which requires C with a vulnerability) are the hard case.

**Policy engines**: OPA (Open Policy Agent) / Rego — a purpose-built policy language for "does this deployment violate our security policy?" Worth understanding conceptually.

**Event stream processing**: Security products ingest events (process start, network connection, file access) at very high volume and match against detection rules in real time. Structurally similar to observability ingestion pipelines.

For now: treat security as an interesting future direction. The observability track is deeper and more immediately actionable.
