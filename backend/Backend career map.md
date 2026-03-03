# Backend Career Map

Your job = Domain × Role. Pick one from each.

Your filter: developer-facing products only. Every domain listed below satisfies this.

---

## Axis 1 — Domain

Ranked by realistic job market size for a Go backend developer.

### Tier 1 — Large Job Market

**Observability** Build systems that collect, store, and query metrics, logs, and traces.

You work with: time-series databases, stream processing, compression algorithms, query languages (PromQL, LogQL), high-throughput ingestion (HTTP/gRPC), distributed hash rings for sharding, storage engines optimized for append-heavy writes.

You need to understand: time-series data structures, cardinality (millions of unique label combinations exploding storage), sampling strategies, building query engines that scan terabytes fast, distributed coordination between ingesters/compactors/queriers.

What makes it hard: scale. Everything works at 1,000 metrics. At 10 million metrics with 50 label dimensions, storage explodes, queries time out, ingesters run out of memory.

Product engineer: Build the ingestion pipeline, query engine, alerting evaluator, storage backend — the product customers pay for. Platform engineer: Build the CI/CD that deploys the observability product across 30 regions, internal test environments, capacity planning automation.

Companies: Grafana Labs (remote, ~1000 people, 30 days PTO, all Go), Datadog (hybrid, ~6000 people, Go/Python/Java), Chronosphere, Honeycomb, VictoriaMetrics, SigNoz, Elastic.

Real job title: "Senior Backend Engineer" at Grafana Labs — requires Go, distributed systems, relational databases. $154k-$185k USD. On-call included but structured.

---

**Security** Build scanning engines, detection systems, identity platforms, policy enforcement tools.

You work with: vulnerability databases (CVE/NVD), container image formats (OCI layers), AST parsers for source code scanning, event stream processing, rule/policy engines (Rego), cryptography libraries, RBAC systems.

You need to understand: how container images are structured internally (layers, manifests), how dependency trees work across package managers (npm, pip, go modules), how to match patterns in event streams for threat detection, how policy languages work, supply chain security (SBOM, provenance).

What makes it hard: accuracy. Too many false positives and users ignore your scanner. Miss a real vulnerability and you're useless. Speed matters too — scanning must fit into CI pipelines without slowing developers down.

Product engineer: Build the scanner that unpacks container images and checks dependencies, or the detection engine processing millions of security events, or the policy engine that enforces rules across cloud accounts. Platform engineer: Build the systems that keep vulnerability databases synced, the testing infrastructure, the deployment pipeline for the security product.

Companies: CrowdStrike, Aqua Security, Snyk, Wiz, SentinelOne, Panther Labs.

Note: Engineering overlaps heavily with observability — same patterns (event ingestion, pattern matching, alerting) applied to security data. If you like observability engineering, security is a natural adjacent move.

---

**Platforms / PaaS** Build deployment platforms — developer pushes code, it runs in production.

You work with: container orchestration, build systems, load balancers, TLS/certificate management, DNS configuration, deployment strategies (blue-green, canary), autoscaling logic, multi-region routing, billing/metering.

You need to understand: the full deployment journey (push code → build image → schedule → configure networking → TLS → DNS → route traffic), how autoscaling decisions work, how deployment rollbacks work, how to hide enormous infrastructure complexity behind a simple API/CLI.

What makes it hard: seamlessness. The developer expects `git push` to just work. Behind that is networking, TLS, DNS propagation, health checks, rollback logic, multi-region coordination. The engineering is hiding complexity.

Product engineer: Build the deployment pipeline, the scaling system, the networking layer, the CLI/API developers interact with. Platform engineer: Build the internal systems that operate the PaaS itself — the multi-region deployment of the platform, internal observability, incident automation.

Companies: Fly.io (Go/Rust, startup), Vercel (Go backend, ~500 people), Railway, Render, Netlify.

---

### Tier 2 — Medium Job Market

**Provisioning / IaC** Build tools that define and manage infrastructure through code.

You work with: graph algorithms (dependency resolution), state machines, diffing algorithms (desired state vs actual state), cloud provider APIs (AWS/GCP/Azure SDKs), plugin architectures (gRPC between core and providers), configuration parsing.

You need to understand: how declarative config becomes an execution plan (parse → build dependency graph → diff against current state → determine create/update/delete order), how state management works and why it's the hardest problem in IaC, how plugin systems work.

What makes it hard: correctness. If your diff says "no changes" but there are changes, someone's infrastructure is wrong. If execution order is wrong, resources get created that can't be deleted.

Product engineer: Work on the core engine (state management, planning, graph execution) or — more commonly — build and maintain providers that translate cloud APIs into the IaC model. Most job openings are provider work, not core engine work. Platform engineer: Build the provider registry, collaboration features, CI that tests thousands of provider combinations.

Companies: HashiCorp/IBM (Terraform, Vault, Consul — all Go), Pulumi.

Note: HashiCorp was acquired by IBM in 2024. Job market is real but concentrated in fewer companies than observability or security.

---

**Networking / Edge** Build systems that route, filter, and accelerate network traffic.

You work with: HTTP/2 and HTTP/3 (QUIC) protocol internals, reverse proxies, TLS termination, DNS resolvers, WebAssembly runtimes (for edge computing), CDN caching/invalidation, connection pooling at scale.

You need to understand: protocol-level networking (how HTTP, TLS, DNS actually work under the hood), how reverse proxies route and transform requests, how edge computing runtimes sandbox user code safely (V8 isolates, Wasm), how CDN caching works.

What makes it hard: latency. Your code sits in the request path of every user of every customer. Adding 1ms of overhead to a proxy serving millions of sites is a disaster. The engineering is moving data as fast as possible while still doing useful work on it.

Product engineer: Build the proxy, the edge runtime, the DNS resolver, the caching layer — the product customers use. Platform engineer: Build the systems that deploy software to hundreds of edge locations, configuration management across global PoPs, testing infrastructure.

Companies: Cloudflare (large, public, significant Go usage), Fastly, Cilium/Isovalent.

Note: This is more protocol/systems-heavy than the other domains. Networking knowledge is a steeper learning curve if you don't already have it.

---

**Storage / Databases** Build distributed databases, storage engines, object stores.

You work with: B-trees, LSM trees, write-ahead logs, Raft consensus protocol, SQL parsers, query optimizers, memory management, disk I/O, MVCC (multi-version concurrency control), serializable transactions.

You need to understand: how data gets written to disk efficiently (batching, WAL, fsync), how indexes accelerate reads (bloom filters, page caches), how distributed consensus works (Raft leader election, log replication), how query optimization plans execution.

What makes it hard: correctness under failure. If your database loses data or returns wrong results during a node crash, that's catastrophic. Most technically demanding domain on this list.

Product engineer: Work on the storage engine, query optimizer, replication layer, consensus protocol. Platform engineer: Build testing infrastructure (simulating network partitions), benchmarking systems, managed database deployment automation.

Companies: CockroachDB, Minio, Dgraph, InfluxData, PingCAP.

Note: Highest bar to entry. These companies want distributed systems theory knowledge, not just Go backend experience. Your 2.5 years wouldn't be enough without significant self-study. Realistic as a long-term direction, not a first target.

---

### Tier 3 — Small/Unrealistic for Now

**Runtime / Containers** Docker and containerd are mature products. The number of companies hiring people to work on container runtimes is very small. This is Linux kernel boundary work (namespaces, cgroups, syscalls). Not realistic as a job target — almost no open positions.

**Orchestration (Kubernetes core)** Working on Kubernetes itself is a tiny group at a few companies. What's actually common is building _on top of_ Kubernetes (writing operators, Helm charts) — but that's platform engineering at any company, not a dedicated domain. Don't target this as a domain.

**AI/ML Infrastructure** Go is not the primary language here. Most ML infra is Python and C++. Go appears in orchestration and API layers but you'd be swimming against the current targeting this as a Go developer. The space is also moving so fast that tooling changes every 6 months.

---

## Axis 2 — Role

What you do. Filtered to the two relevant to you.

| Role                  | What it means                                                                              | Your fit                                                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Product Engineer**  | Build what the company sells. External-facing. What the product is depends on the company. | 2.5 years experience in this role, but at non-developer-facing companies. Your strongest role type.                                   |
| **Platform Engineer** | Build internal systems that help other engineers at your company ship. Internal-facing.    | No direct experience but transferable skills. Fastest growing role title. Exists at every mid-to-large company, not just cloud/infra. |

Roles not included: Infrastructure Engineer and SRE — you said you don't want these.

---

## Your Target

**Primary:** Product Engineer at an Observability company **Secondary:** Product Engineer at a Security or Platforms/PaaS company **Backup:** Platform Engineer at any tech company (universal demand, not domain-specific)

---

## Not Domains (Reference)

Things that are delivery formats or skills, not domains you can target:

- CLI tools — built within every domain
- APIs, SDKs — built within every domain
- Data pipelines, real-time systems — skills used across domains
- "Developer tools" — describes the entire cloud/infra space
- Microservices — architecture pattern

---

## Next Steps

1. Read 2-3 posts from Grafana Labs engineering blog — see if observability problems pull you
2. Read 2-3 posts from Fly.io blog — compare PaaS engineering feel
3. Search job boards: "Backend engineer" + Grafana / Datadog / observability
4. Apply broadly to Go backend roles, aim at Tier 1 domains
5. Specialize on the job

Backend is the vehicle. Game development is the destination.