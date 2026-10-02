# practice.md — Hands-on Labs

Every lab is a **real-world scenario** you can run on your laptop with Docker Compose.
Each lab links back to videos in `learn.md`. Do the lab in the same week you watch them.

## Rules

1. **Hypothesis before code.** Write what you expect, with numbers.
2. **Smallest setup that answers the question.** No Kubernetes until you need it.
3. **Always break it.** Kill a container, add latency, push 10x load.
4. **Numbers or it didn't happen.** Results go to `labs/NN-name/RESULTS.md`.
5. **One ADR per real decision** in `decisions/`.
6. **AI may scaffold, never decide.** You design the experiment; AI can help write boilerplate (see `CLAUDE.md`).

## Toolbox

| Need | Tool |
|---|---|
| Run everything locally | Docker + Docker Compose |
| Load testing | k6 (or `pgbench` for Postgres-only tests) |
| Relational DB | PostgreSQL (+ `pgvector` for the capstone) |
| Document DB | MongoDB (replica sets, sharded cluster) |
| Leaderless DB | Cassandra or ScyllaDB |
| Cache / rate limit | Redis |
| Load balancer | Nginx or HAProxy |
| Events | Kafka (KRaft mode) or Redpanda |
| Fault injection | Toxiproxy, `docker stop`, `docker pause` |
| Observability | OpenTelemetry + Prometheus + Grafana + Jaeger/Tempo |
| Frontend | Lighthouse, Chrome DevTools Performance, axe |

---

## Lab template (copy for each lab)

```markdown
## Lab N — <title>
- Linked videos: #..
- Real-world scenario:
- Hypothesis (with numbers):
- Setup (components + versions):
- Steps:
- Break it:
- Metrics collected:

| Variant | RPS | p50 | p95 | p99 | Errors | Notes |
|---|---|---|---|---|---|---|
| baseline | | | | | | |
| change 1 | | | | | | |

- Prediction vs reality — what surprised me and why:
- What I'd do differently in production:
- ADR written: decisions/ADR-XXX.md
- Time spent:
```

---

## Lab 0 — Learn to measure (videos #5, #6)
**Scenario:** Your manager asks "is the API fast enough?" You need a real answer.
- Build a tiny HTTP service with one endpoint that sleeps a random 5–50 ms.
- Load test it with k6 at 10, 50, 200 virtual users.
- Record p50, p95, p99 and throughput. Plot latency vs. load.
- **Break it:** add a 1-in-100 request that takes 2 s. Watch the average barely move while p99 explodes.
- **Takeaway to write down:** why averages lie, and what "reliable, scalable, maintainable" means in numbers.

## Lab 1 — Scale a URL shortener from zero (videos #1–4)
**Scenario:** Your link-shortener goes viral on social media overnight.
Run each stage and load test it with the same k6 script (90% reads, 10% writes):
1. App + DB in one container.
2. Separate the database.
3. Two app instances behind Nginx (stateless app!).
4. Add Redis cache-aside for `GET /:code`.
5. Add a Postgres read replica for reads that miss the cache.
- **Break it:** kill one app instance during the test; flush Redis mid-test (cache stampede); stop the replica.
- **Measure:** RPS and p95 per stage, DB CPU, cache hit ratio.
- **ADR:** cache TTL and invalidation strategy.

## Lab 2 — Pick the right data model (videos #7, #8)
**Scenario:** An e-commerce store with customers, orders, items, and "customers who bought X also bought Y".
- Model it in Postgres (normalized) and in MongoDB (orders as documents).
- Query A: "show order #123 with its items and customer" — compare code and speed.
- Query B: "products often bought together" — notice where each model struggles (many-to-many).
- **Takeaway:** write when you'd pick document vs relational vs graph.

## Lab 3 — Indexes for real (videos #9–12)
**Scenario:** The orders page takes 4 seconds in production.
- Generate ~5M rows in Postgres with `generate_series`.
- Run `EXPLAIN (ANALYZE, BUFFERS)` for `WHERE customer_id = ? ORDER BY created_at DESC LIMIT 20`.
- Add: single index → composite `(customer_id, created_at)` → covering index with `INCLUDE`. Record each plan and timing.
- Test composite column order: swap the columns and see which queries stop using the index.
- **Write cost:** measure insert throughput with 0, 2, and 5 indexes.
- **LSM side:** ingest the same write-heavy stream into Cassandra/ScyllaDB and Postgres; compare write throughput and read latency.
- **Takeaway:** a rule of thumb for "should I add this index?"

## Lab 4 — Encoding & schema evolution (videos #13, #14)
**Scenario:** Two teams deploy services at different times, and messages must keep working.
- Encode the same 1,000 order events as JSON, Protobuf, and Avro. Compare size and encode/decode time.
- Evolve the schema: add an optional field, remove a field, rename a field.
- Check: can an **old** consumer read **new** messages (forward compatibility)? Can a **new** consumer read **old** messages (backward)?
- **ADR:** which format for internal service-to-service events and why.

## Lab 5 — Replication & consistency (videos #15–19)
**Scenario:** Users update their profile, refresh the page, and see the old data.
- Postgres primary + 2 streaming replicas.
- Write to the primary, read from a replica immediately → reproduce the **read-your-writes** bug.
- Fix it 3 ways: read from primary after write; session stickiness; wait for replica LSN. Compare cost.
- Measure replication lag under load; compare async vs. synchronous replication write latency.
- **Break it:** stop the primary, promote a replica, see what writes were lost (async).
- **Leaderless:** 3-node Cassandra with replication factor 3. Read/write at `ONE` vs `QUORUM`; stop a node; observe stale reads and availability.
- **Takeaway:** explain W + R > N with your own measured example.

## Lab 6 — Sharding (videos #20, #21)
**Scenario:** One database can't hold all of a chat app's messages anymore.
- Follow the MongoDB sharded cluster demo from video #21.
- Shard by a **bad key** (timestamp) → observe the hot shard. Then by hashed `user_id` → compare distribution.
- Run a query that doesn't include the shard key → observe scatter-gather.
- (Optional) Repeat with Postgres + Citus.
- **ADR:** shard key choice for messages.

## Lab 7 — Inside the database: MVCC & isolation (videos #22, #23)
**Scenario:** Two people book the last seat at the same time; a bank balance goes wrong.
- Open two `psql` sessions.
- **Lost update** at `READ COMMITTED` with read-modify-write in app code → fix with `SELECT ... FOR UPDATE` or an atomic `UPDATE ... SET x = x - 1`.
- **Write skew** (two doctors both go off call) is allowed under `REPEATABLE READ` → prevented by `SERIALIZABLE` (handle the retry!).
- Update a table heavily, then look at dead tuples and run `VACUUM`; watch table size.
- Compare: same booking logic in MongoDB with multi-document transactions.

## Lab 8 — Monolith → microservices (videos #24–27)
**Scenario:** An order flow: `orders`, `payments`, `inventory`.
- Build it first as a **modular monolith** with clear module boundaries.
- Extract `payments` into its own service. Measure the latency added by the network hop.
- Compare REST/JSON vs gRPC/Protobuf for the same call (latency, payload size, developer effort).
- **Break it:** make payments slow (Toxiproxy +500 ms) → see the whole checkout slow down. Add timeouts.
- **ADR:** what you would keep in the monolith and why.

## Lab 9 — Events, transactions & CQRS (videos #28–31)
**Scenario:** "Order placed" must reliably trigger payment, inventory, and email — without double charging.
- Kafka with a `orders` topic (3 partitions), keyed by `order_id`.
- Implement the **outbox pattern** (write order + event in one DB transaction; a relay publishes it).
- Make consumers **idempotent** with a processed-event table; replay the topic and prove no double charge.
- Implement a **saga**: payment fails → release inventory → cancel order.
- **Break it:** kill a consumer mid-batch; restart; check for duplicates and lost events.
- Build a **CQRS read model** (order history view) from events; measure how stale it gets.

## Lab 10 — Resilience & durable execution (video #32)
**Scenario:** A third-party payment API is flaky.
- With Toxiproxy, inject latency, timeouts, and connection resets.
- Add: timeouts → retries with exponential backoff + jitter → circuit breaker. Measure error rate and p99 at each step.
- Show how naive retries amplify load (retry storm).
- (Optional) Rebuild the saga from Lab 9 with a durable execution engine (Restate or Temporal) and compare code size and failure handling.

## Lab 11 — API security (videos #33, #34)
**Scenario:** A pentest report lands on your desk.
- Reproduce **BOLA**: user A fetches `/orders/{id}` belonging to user B. Fix it with ownership checks + a test.
- Implement rate limiting (token bucket in Redis) and test it with k6.
- Check JWT handling: expired tokens, wrong algorithm, missing audience.
- Add secret scanning and dependency scanning to CI. Write a 1-page SSDLC checklist for your labs.

## Lab 12 — Observability (videos #35, #36)
**Scenario:** "Checkout is slow sometimes" — find out why in under 10 minutes.
- Instrument Lab 9 services with OpenTelemetry; send traces to Jaeger/Tempo and metrics to Prometheus.
- Build a Grafana dashboard with RED metrics (Rate, Errors, Duration) per service.
- Define one SLO (e.g., 99% of checkouts < 800 ms) and an alert.
- **Break it:** inject latency into one dependency without telling yourself which (use a script that picks randomly). Find it using only traces and dashboards. Time yourself.

## Lab 13 — Interview simulations (videos #37–40)
**Scenario:** Real interview conditions.
- For each problem (stock price tracker, YouTube payout system): 45-minute timer, write a design doc (requirements, estimates, API, data model, diagram, scaling, failures). Then watch the video and diff your design.
- Use the **mock interviewer prompt** from `README.md` for at least 3 extra problems (chat app, rate limiter, notification system).
- **Build one component for real:** an idempotent payout ledger (double-entry, no double payouts on retry) with tests.
- **LLD:** implement the vending machine as a state machine with unit tests for every transition.

---

## Capstone — An AI-ready production system
**Scenario:** "Ask our docs" — a Q&A service over company documents.

```
Client → API (rate limited, auth) → queue → ingestion workers → Postgres + pgvector
                     │
                     └→ query service → retrieval → LLM call → response cache
                                  (timeouts, retries, cost tracking, tracing)
```

Must-haves:
- Async ingestion pipeline (Kafka or a simple queue) with idempotent workers.
- Vector search with `pgvector`; measure retrieval latency vs. number of documents.
- LLM calls with timeouts, retries, a response cache, and **per-request token cost** logged.
- An **eval set** of 30 questions with expected answers; run it on every change.
- OpenTelemetry traces covering retrieval + model calls; a dashboard for latency, cost, and error rate.
- (Optional) Expose the service as an **MCP server** so AI agents can use it; think about authorization.
- Write a full design doc + 3 ADRs. This is your portfolio piece.

---

## Frontend labs

### F1 — Performance budget (video F5)
- Take a real page (yours or a demo app). Record LCP, INP, CLS with Lighthouse and DevTools.
- Apply one change at a time: image optimization, code splitting, lazy loading, removing a heavy dependency. Record each delta.

### F2 — Rendering strategies (videos F2, F4)
- Build the same product page with client-side rendering and server-side rendering (or static generation).
- Compare TTFB, LCP, and JS bundle size on a throttled "slow 4G" profile.

### F3 — GraphQL vs REST (video F6)
- Same screen with REST (multiple calls) and GraphQL (one query).
- Count requests and payload size; reproduce the N+1 problem on the server and fix it with a DataLoader.

### F4 — TypeScript migration (video F3)
- Migrate a small JS module to TypeScript with `strict` on. Log every bug the compiler found.

### F5 — Accessibility (video F7)
- Run axe on your app, then use it **keyboard-only** and with a screen reader for 10 minutes. Fix the top 5 issues.

---

## Progress log

| Week | Videos | Lab | Key number I measured | One-line lesson |
|---|---|---|---|---|
| 1 | #1–5 | Lab 0, Lab 1 | | |
| 2 | | | | |
