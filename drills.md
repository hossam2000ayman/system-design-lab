# drills.md — One Hands-on Drill per Concept

`practice.md` has big labs (2–6 hours, real-world scenarios).
This file has **small drills (20–60 minutes) for each single concept**, so you
*feel* the idea once, and then re-do it over time so it never fades.

> Theory tells you **what**. A drill shows you **why** — because you watched it break.

---

## How to remember forever: the re-drill schedule

| When | What you do | Time |
|---|---|---|
| **Day 0** | Do the full drill step by step. Write the "aha" in your own words. | 20–60 min |
| **Day 1** | Without running anything: write what you expect to see at each step. | 5 min |
| **Day 7** | Answer the recall questions out loud, no notes. Draw the diagram from memory. | 10 min |
| **Day 30** | **Speed-run:** rebuild the drill from an empty folder, no notes, no AI. | 15–20 min |
| **Day 90** | **Twist:** change one thing (bigger data, a different failure, another tool) and predict the result before running. | 20 min |

Track it at the bottom of this file. If a speed-run fails, the concept goes back to Day 0. That's normal and it's exactly how memory gets stronger.

### Every drill follows the same shape
1. **Idea** — one sentence.
2. **Setup** — the smallest thing to run.
3. **Steps** — do, then observe.
4. **Break it** — make it fail on purpose.
5. **Aha** — what you should notice.
6. **Remember it as** — a short hook.
7. **Recall questions** — for Day 7.

---

## Shared setup (do once)

```bash
# PostgreSQL
docker run -d --name pg -e POSTGRES_PASSWORD=pw -p 5432:5432 postgres:16
docker exec -it pg psql -U postgres          # open a SQL shell

# Redis
docker run -d --name redis -p 6379:6379 redis:7
docker exec -it redis redis-cli

# k6 (load testing) — install from https://k6.io or run it with Docker
k6 version
```

Open **two terminals** for any drill that says "Session A / Session B".

---

# Part 1 — Performance & scaling

## D1. Latency vs throughput, and why p99 matters
**Idea:** the average hides the slow requests your users actually feel.

**Setup:** `server.py`
```python
import random, time
from http.server import ThreadingHTTPServer, BaseHTTPRequestHandler

class H(BaseHTTPRequestHandler):
    def do_GET(self):
        # 1% of requests are very slow, the rest take 5–50 ms
        time.sleep(2 if random.random() < 0.01 else random.uniform(0.005, 0.05))
        self.send_response(200); self.end_headers(); self.wfile.write(b"ok")
    def log_message(self, *a): pass

ThreadingHTTPServer(("0.0.0.0", 8000), H).serve_forever()
```
`load.js`
```js
import http from 'k6/http';
export const options = {
  vus: 50, duration: '30s',
  summaryTrendStats: ['avg', 'med', 'p(95)', 'p(99)', 'max'],
};
export default function () { http.get('http://localhost:8000/'); }
```
**Steps:** `python server.py`, then `k6 run load.js`. Write down avg, p95, p99, and requests/sec.
Run again with `vus: 10` and `vus: 200`. Fill this table:

| VUs | req/s | avg | p95 | p99 |
|---|---|---|---|---|

**Break it:** change `0.01` to `0.05` (5% slow requests).
**Aha:** the average looks fine while p99 is ~2 s. A user who loads a page that makes 50 requests will almost always hit one slow request.
**Remember it as:** *"Average is for reports, p99 is for users."*
**Recall:** Why can throughput increase while latency gets worse? Which percentile would you put in an SLO and why?

## D2. Stateless services & horizontal scaling
**Idea:** you can only add more servers freely if no server keeps important data in its memory.

**Setup:** a `/count` endpoint that increments an **in-memory** counter. Run 2 copies behind Nginx:
```nginx
events {}
http {
  upstream app { server app1:8000; server app2:8000; }
  server { listen 80; location / { proxy_pass http://app; } }
}
```
**Steps:** call `/count` 10 times. The numbers jump around (1, 1, 2, 2, 3…) because each server has its own counter.
**Fix:** replace the in-memory counter with Redis `INCR counter`. Now the count is correct no matter which server answers.
**Break it:** `docker stop app1` during a k6 test. With the Redis version, nothing is lost.
**Aha:** state belongs in a shared store (DB, Redis); servers become disposable.
**Remember it as:** *"Cattle, not pets."*
**Recall:** Where do sessions live in a horizontally scaled app? What's the downside of sticky sessions?

## D3. Load balancing & consistent hashing
**Idea:** when a server is added or removed, consistent hashing moves only a small part of the keys.

**Setup:** `hashing.py`
```python
import hashlib, bisect
def h(x): return int(hashlib.md5(x.encode()).hexdigest(), 16)

keys = [f"user{i}" for i in range(100_000)]

def modulo(nodes): return {k: nodes[h(k) % len(nodes)] for k in keys}

def ring(nodes, vnodes=100):
    points = sorted((h(f"{n}#{v}"), n) for n in nodes for v in range(vnodes))
    hashes = [p[0] for p in points]
    def owner(k):
        i = bisect.bisect(hashes, h(k)) % len(points)
        return points[i][1]
    return {k: owner(k) for k in keys}

def moved(a, b): return sum(a[k] != b[k] for k in keys) / len(keys)

four, three = ["A", "B", "C", "D"], ["A", "B", "C"]
print("modulo moved:", moved(modulo(four), modulo(three)))
print("ring moved:  ", moved(ring(four), ring(three)))
```
**Steps:** run it. Expect ~75% moved with modulo vs ~25% with the ring.
**Twist:** set `vnodes=1` and count keys per node — see how uneven it gets.
**Also try in Nginx:** `least_conn;` vs default round robin when one backend is slow (add `time.sleep` to one app), compare p95.
**Aha:** with modulo hashing, removing one cache server invalidates almost the whole cache.
**Remember it as:** *"Modulo moves everything, the ring moves your share."*
**Recall:** Why virtual nodes? When is round robin a bad choice?

## D4. Caching: cache-aside, TTL, and stampede
**Idea:** a cache makes reads fast until many keys expire at once and the DB gets hammered.

**Setup:** a small API with `GET /product/:id` that runs a slow query (`SELECT pg_sleep(0.2), ...`). Count DB calls with a counter or watch `SELECT count(*) FROM pg_stat_activity WHERE state='active';`.
**Steps:**
1. No cache → measure p95 with k6.
2. Cache-aside: check Redis → if missing, read DB → `SET key value EX 30`. Measure again and record the hit ratio.
3. Update the product in the DB → the API still shows the old value until the TTL ends. That's stale data you chose to accept.

**Break it:** run 200 VUs and run `FLUSHALL` in Redis. Watch DB calls spike (stampede).
**Fix it:** add a lock (`SET lock:key 1 NX EX 5` — only the winner queries the DB) or add random jitter to TTLs.
**Aha:** a cache adds a consistency problem and a failure mode, not only speed.
**Remember it as:** *"Every cache is a promise to be a little bit wrong."*
**Recall:** Cache-aside vs write-through? Two ways to stop a stampede?

## D5. Back-of-the-envelope estimation
**Idea:** quick math tells you if you need 1 server or 100 before you build anything.

**Steps:**
1. 10M daily active users × 20 requests/day = 200M requests/day.
2. 200M ÷ 86,400 s ≈ **2,300 req/s average**; × 3 for peak ≈ **7,000 req/s**.
3. Take the req/s **you measured** for one app instance in D1/Lab 1. Divide → number of instances needed (+ spare capacity).
4. Storage: 1M new rows/day × 1 KB = 1 GB/day ≈ 365 GB/year, before indexes and replicas.

**Aha:** your own measurements turn guesses into capacity plans.
**Remember it as:** *"86,400 seconds in a day — divide first, panic later."*
**Recall:** Estimate QPS and storage for a URL shortener with 100M new links/month.

---

# Part 2 — Data storage

## D6. Data models: relational vs document vs graph
**Idea:** documents are great for data read together; relations are great for data shared everywhere.

**Steps (Postgres only):**
1. Store orders as `JSONB` documents that embed the product name. Store the same data in normalized tables (`orders`, `order_items`, `products`).
2. Read "order #123 with items": compare the query complexity.
3. Rename a product that appears in 1,000 orders. Normalized: 1 row update. JSONB: update 1,000 documents.
4. Graph taste: a `friends(a, b)` table. Find friends-of-friends-of-friends with a recursive CTE and look at how the query grows. (Optional: the same in Neo4j with one Cypher line.)

**Aha:** duplication makes reads easy and updates painful.
**Remember it as:** *"Embed what you read together, reference what you share."*
**Recall:** Which model for a social network? For a product catalog with variable attributes?

## D7. How an index works (B-tree)
**Idea:** an index turns "scan everything" into "walk down a short tree".

**Setup:**
```sql
CREATE TABLE orders (
  id bigserial PRIMARY KEY, customer_id int, status text,
  total numeric, created_at timestamptz);

INSERT INTO orders (customer_id, status, total, created_at)
SELECT (random()*100000)::int,
       (ARRAY['new','paid','shipped'])[1 + floor(random()*3)::int],
       random()*500,
       now() - random() * interval '365 days'
FROM generate_series(1, 5000000);
ANALYZE orders;
```
**Steps:**
1. `EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 20;` → note **Seq Scan** and the time.
2. `CREATE INDEX idx_c ON orders(customer_id);` → run again → **Index Scan** or Bitmap scan + a Sort.
3. `CREATE INDEX idx_cc ON orders(customer_id, created_at);` → the sort disappears.
4. `SELECT * FROM orders WHERE created_at > now() - interval '1 day';` → the composite index isn't used well because `created_at` isn't the leading column.
5. Look inside the tree:
   ```sql
   CREATE EXTENSION pageinspect;
   SELECT level FROM bt_metap('idx_cc');   -- tree height
   ```
   5M rows and the tree is only ~2–3 levels deep.

**Break it:** time inserting 100k rows with 0 extra indexes vs 4 extra indexes.
**Aha:** each index speeds up some reads and slows down every write. Column order matters.
**Remember it as:** *"Index = phone book: sorted by last name first, then first name."*
**Recall:** Why can't `(a, b)` help a query on `b` alone? Cost of too many indexes?

## D8. Covering index & index-only scan
**Steps:**
```sql
CREATE INDEX idx_cov ON orders(customer_id, created_at) INCLUDE (total);
VACUUM orders;   -- updates the visibility map so index-only scans work
EXPLAIN ANALYZE SELECT created_at, total FROM orders WHERE customer_id = 42;
```
Look for **Index Only Scan** and `Heap Fetches: 0`. Then `SELECT *` → back to a normal index scan.
**Aha:** if the index contains every column you need, the table isn't touched at all.
**Remember it as:** *"Don't open the book if the index already has the answer."*

## D9. LSM-tree (why NoSQL writes are fast)
**Idea:** write to memory, flush sorted files, merge later. Fast writes, more work on reads.

**Setup:** build a toy LSM in ~40 lines of Python:
- `put(k, v)` → writes to a dict (memtable).
- When the memtable has 1,000 keys → write it **sorted** to `sst_N.json` and clear it.
- `get(k)` → check memtable, then SSTables **newest first**.
- `compact()` → merge all SSTables into one, keeping the newest value per key.

**Steps:** insert 100k keys with many overwrites. Count how many files a `get` checks before and after compaction. Delete a key → you need a **tombstone** marker, not a real delete.
**Aha:** writes are just appends; reads and compaction pay the price.
**Remember it as:** *"LSM: write now, clean up later. B-tree: clean up on every write."*
**Recall:** What is write amplification? Why do LSM databases use Bloom filters?

## D10. Encoding & schema evolution
**Idea:** old and new versions of services must understand each other's messages.

**Steps:**
1. `pip install protobuf grpcio-tools`. Write `order.proto` v1: `id = 1; total = 2;`.
2. Encode 1,000 orders as JSON and as Protobuf; compare bytes.
3. v2: add `string coupon = 3;`. Encode with v2, decode with v1 → works (unknown field skipped). Encode with v1, decode with v2 → works (`coupon` is empty).
4. **Break it:** in v2 reuse number `2` for a different type. Decode old data → garbage or an error.

**Aha:** field **numbers** are the contract, not names.
**Remember it as:** *"Never reuse a tag number."*
**Recall:** Forward vs backward compatibility — which one matters when consumers deploy later than producers?

---

# Part 3 — Distributed data

## D11. Replication lag & read-your-writes
**Idea:** a replica is a slightly older copy of the primary.

**Setup:** a Postgres primary + 1 streaming replica in Docker Compose. Key steps:
- On the primary, allow replication connections in `pg_hba.conf` (`host replication all all scram-sha-256`).
- Create the replica with `pg_basebackup -h primary -U postgres -D <data dir> -R -X stream`, then start it.
- *(OK to ask AI to scaffold this compose file — but you write the steps above first.)*

**Steps:**
1. On the replica, add an artificial delay:
   ```sql
   ALTER SYSTEM SET recovery_min_apply_delay = '5s';
   SELECT pg_reload_conf();
   ```
2. Insert a row on the primary, then immediately `SELECT` it on the replica → **missing**. After 5 s → there.
3. Measure lag on the replica: `SELECT now() - pg_last_xact_replay_timestamp();`
4. Fix "read-your-writes": after a user writes, read their own data from the primary for a few seconds.

**Break it:** stop the primary during writes. With async replication, the last writes may never reach the replica.
**Aha:** "eventually consistent" means "your user just saw old data".
**Remember it as:** *"Replicas live in the past."*
**Recall:** Three ways to give users read-your-writes? Sync vs async replication trade-off?

## D12. Multi-leader conflicts
**Idea:** two leaders accept writes at the same time → conflicts you must resolve.

**Steps (Python simulation):** two dicts `dc_eu` and `dc_us`. Both set `title` of doc 1 while "offline". Merge with last-write-wins by timestamp → one edit disappears silently. Then merge by keeping both values (like a shopping cart union) → no loss, but the app must handle it.
**Aha:** last-write-wins = silent data loss.
**Remember it as:** *"Two leaders, two truths."*

## D13. Quorums (W + R > N)
**Idea:** if writes and reads overlap on at least one replica, you read the latest value.

**Setup:** 3-node Cassandra (`cassandra:4.1`, give nodes a few minutes to join). In `cqlsh`:
```sql
CREATE KEYSPACE lab WITH replication = {'class':'SimpleStrategy','replication_factor':3};
CREATE TABLE lab.kv (k text PRIMARY KEY, v text);
CONSISTENCY QUORUM;
```
**Steps:**
1. Write and read with `QUORUM` (2 of 3). `docker stop` one node → still works.
2. Stop a second node → `QUORUM` fails, `CONSISTENCY ONE` still works.
3. Turn on `TRACING ON` and compare latency of `ONE` vs `QUORUM`.

**Aha:** you choose per query between being correct and being available/fast.
**Remember it as:** *"Overlap = truth."* (N=3, W=2, R=2 → 2+2 > 3)
**Recall:** What happens with W=1, R=1? Why do quorums cost latency even without failures?

## D14. CAP / PACELC
**Idea:** during a network split you choose consistency **or** availability; without a split you still choose latency **or** consistency.

**Steps:** with D13's cluster, `docker network disconnect <net> <node>` to cut one node off.
- With `QUORUM`, the isolated side refuses requests → consistency chosen.
- With `ONE`, it keeps answering with possibly stale data → availability chosen.
- Reconnect, and you've already measured the "Else" part (latency vs consistency) in D13 step 3.

**Remember it as:** *"Partition? Pick C or A. Else? Pick L or C."*

## D15. Sharding & hot partitions
**Idea:** a bad shard key sends most traffic to one machine.

**Steps (Python first):** simulate 100k messages over 4 shards:
- Range by timestamp (each shard owns one month) → all new writes land on the last shard.
- Hash of `user_id` → even spread.
- Now add one "celebrity" user with 30% of the messages → even hashing gets a hot spot.

Then do the real thing with the MongoDB sharded cluster from video #21: run `sh.status()` and `db.collection.getShardDistribution()`.
**Aha:** the shard key decides your scalability more than the hardware does.
**Remember it as:** *"Pick the key by how you write AND how you read."*
**Recall:** What's scatter-gather? How do you handle a celebrity key?

---

# Part 4 — Transactions & databases internals

## D16. Lost update
**Setup:**
```sql
CREATE TABLE acct (id int PRIMARY KEY, balance int);
INSERT INTO acct VALUES (1, 100);
```
**Steps (READ COMMITTED, the Postgres default):**
| Session A | Session B |
|---|---|
| `BEGIN;` | `BEGIN;` |
| `SELECT balance FROM acct WHERE id=1;` → 100 | `SELECT balance FROM acct WHERE id=1;` → 100 |
| `UPDATE acct SET balance=90 WHERE id=1; COMMIT;` | |
| | `UPDATE acct SET balance=80 WHERE id=1; COMMIT;` |

Final balance: **80**, but two withdrawals of 10 and 20 should give **70**. One update was lost.
**Fix 1:** `UPDATE acct SET balance = balance - 20 WHERE id=1;` (atomic).
**Fix 2:** `SELECT ... FOR UPDATE` → Session B waits.
**Fix 3:** run both as `BEGIN ISOLATION LEVEL REPEATABLE READ;` → B gets *could not serialize access due to concurrent update* and must retry.
**Remember it as:** *"Read-then-write in app code is a race."*

## D17. Write skew
**Setup:**
```sql
CREATE TABLE doctors (name text PRIMARY KEY, on_call bool);
INSERT INTO doctors VALUES ('alice', true), ('bob', true);
```
**Steps:** both sessions `BEGIN ISOLATION LEVEL REPEATABLE READ;`
- A: `SELECT count(*) FROM doctors WHERE on_call;` → 2 → "ok, I can leave"
- B: same → 2
- A: `UPDATE doctors SET on_call=false WHERE name='alice'; COMMIT;`
- B: `UPDATE doctors SET on_call=false WHERE name='bob'; COMMIT;`
- Result: **nobody on call**, and no error.

Repeat with `SERIALIZABLE` → one commit fails with a serialization error. Your app must catch it and retry.
**Aha:** each transaction was correct alone; together they broke the rule.
**Remember it as:** *"Snapshot isolation sees a photo, not the live scene."*

## D18. MVCC & VACUUM
**Steps:**
```sql
CREATE TABLE t AS SELECT g AS id, 0 AS v FROM generate_series(1, 1000000) g;
ALTER TABLE t SET (autovacuum_enabled = false);
SELECT ctid, xmin, xmax, * FROM t WHERE id = 1;
UPDATE t SET v = 1 WHERE id = 1;
SELECT ctid, xmin, xmax, * FROM t WHERE id = 1;    -- new ctid = new row version
SELECT pg_size_pretty(pg_total_relation_size('t'));
UPDATE t SET v = v + 1;                              -- 1M new versions
SELECT pg_size_pretty(pg_total_relation_size('t'));  -- roughly doubled
SELECT n_dead_tup FROM pg_stat_user_tables WHERE relname = 't';
VACUUM t;      -- dead tuples cleaned, space reusable (file doesn't shrink)
VACUUM FULL t; -- rewrites the table and shrinks it (locks the table!)
```
**Aha:** in Postgres an UPDATE is really INSERT new + mark old as dead.
**Remember it as:** *"Postgres never edits, it rewrites and cleans later."*
**Recall:** Why does a long-running transaction cause table bloat?

---

# Part 5 — Services & communication

## D19. Sync vs async
**Idea:** in a synchronous chain, latencies add up and one slow service slows everyone.

**Steps:** three tiny services A → B → C over HTTP. Make C sleep 300 ms → measure A's latency. Stop C → A fails.
Then: A writes a message to a queue (Redis list `LPUSH` / worker `BRPOP`) and returns immediately. Stop the worker → A still responds; messages wait in the queue. Restart the worker → it catches up.
**Aha:** async trades immediate answers for resilience.
**Remember it as:** *"Sync = phone call. Async = voicemail."*

## D20. REST vs gRPC
**Steps:** the same "get order" call with REST/JSON and with gRPC/Protobuf. Measure payload size and latency with 10k calls. Then change a field in the `.proto` and see the compiler catch the client mismatch.
**Aha:** gRPC gives a strict contract and smaller messages; REST is easier to debug with curl and browsers.
**Remember it as:** *"REST for the outside world, gRPC between my own services."* (rule of thumb, not a law)

## D21. Kafka partitions, consumer groups, ordering
**Setup:** Redpanda (Kafka-compatible, single container — pin a version):
```bash
docker run -d --name rp -p 9092:9092 redpandadata/redpanda redpanda start --mode dev-container
docker exec -it rp rpk topic create orders -p 3
```
**Steps:**
1. Produce with keys: `docker exec -it rp rpk topic produce orders -k user1` (type a few lines), then with `-k user2`, `-k user3`.
2. Open 2 terminals: `docker exec -it rp rpk topic consume orders -g billing` → partitions are split between them.
3. Open a 4th consumer in the same group when there are 3 partitions → one consumer is idle.
4. Check: all messages for `user1` arrive in order, always on the same partition.
5. New group `-g analytics` → it reads everything again independently.

**Aha:** order is guaranteed only inside a partition; partitions = maximum parallelism.
**Remember it as:** *"Same key → same partition → same order."*

## D22. Idempotency
**Idea:** retries are guaranteed to happen; make sure doing something twice has the same effect as once.

**Steps:**
```sql
CREATE TABLE payments (idempotency_key text PRIMARY KEY, order_id int, amount int);
```
1. Build `POST /pay` without a key. Send the same request twice in parallel → 2 charges.
2. Require an `Idempotency-Key` header and `INSERT ... ON CONFLICT (idempotency_key) DO NOTHING`; return the stored result for repeats.
3. Replay 100 duplicate requests with k6 → exactly 1 row.

**Aha:** "exactly-once" in real life = at-least-once delivery + idempotent processing.
**Remember it as:** *"Make it safe to press the button twice."*

## D23. Outbox pattern
**Idea:** saving to the DB and publishing an event are two systems; a crash between them loses the event.

**Steps:**
1. Naive: save order → `sys.exit()` → publish event. The order exists, the event never happened.
2. Outbox: in **one DB transaction**, insert the order **and** a row in `outbox`. A separate relay reads `outbox`, publishes, marks it sent.
3. Kill the relay mid-way → restart → it publishes what's left (maybe twice → that's why D22 matters).

**Remember it as:** *"Write the letter and the envelope in the same transaction."*

## D24. Saga (compensation)
**Steps:** an order flow script: reserve stock → charge payment → create shipment. Make payment fail randomly. Implement compensations in reverse: refund → release stock → cancel order. Log every step and verify the final state is always consistent.
**Remember it as:** *"No global undo button — every step brings its own undo."*

## D25. CQRS & event sourcing
**Steps:**
1. Table `events(id, account, type, amount)` — only inserts, never updates.
2. Compute balance by replaying events. Build a `balances` read table from them (the projection).
3. Next week, invent a new report ("deposits per month") and build it from **past** events.
4. Delete the projection → rebuild it from events.

**Aha:** events are the truth; read models are disposable views.
**When NOT to use:** simple CRUD apps — the extra complexity doesn't pay off.
**Remember it as:** *"Store what happened, compute what is."*

---

# Part 6 — Production

## D26. Timeouts, retries, circuit breakers
**Setup:** put Toxiproxy between your app and Redis/Postgres or a fake payment API.
**Steps:**
1. Add a 2 s latency toxic → without a timeout, your app's threads pile up and p99 explodes.
2. Add a 300 ms timeout → fast failures instead of hanging.
3. Add retries **without** backoff at 100 VUs → watch the dependency get 3–4x the traffic (retry storm).
4. Add exponential backoff + jitter, then a circuit breaker that stops calling after N failures.

**Remember it as:** *"Fail fast, retry politely, stop when it's down."*

## D27. Rate limiting
**Steps:**
1. Fixed window in Redis: `INCR user:42:<minute>` + `EXPIRE 60`; reject above 100.
2. Send 100 requests at second 59 and 100 at second 61 → **200 requests in 2 seconds** passed. That's the boundary burst.
3. Implement a token bucket (tokens refill continuously) and repeat → the burst is capped.

**Remember it as:** *"Windows have edges, buckets don't."*

## D28. Broken object-level authorization (BOLA)
**Steps:** API `GET /orders/{id}` with two users' tokens. As user A, request user B's order ID → it works (bug!). Fix: `WHERE id = $1 AND owner_id = $current_user`. Write an automated test that proves A gets `404`/`403`.
**Remember it as:** *"Logged in ≠ allowed."*

## D29. Observability: logs, metrics, traces
**Steps:**
1. Run Jaeger all-in-one; instrument 2–3 of your services with OpenTelemetry (Python: `opentelemetry-instrument` auto-instrumentation).
2. Hide a `sleep(0.4)` in one random function. Find it **only** with the trace view. Time yourself.
3. Add the trace ID to every log line; jump from a log error to its trace.
4. Expose a Prometheus histogram for request duration and graph p95 in Grafana.

**Aha:** metrics tell you *that* something is wrong, traces tell you *where*, logs tell you *why*.
**Remember it as:** *"Metrics alarm, traces locate, logs explain."*

---

# Part 7 — AI-native systems

## D30. LLM calls as a system component
**Setup:** a local model with Ollama (no API cost), e.g. `ollama run llama3.2 --verbose`.
**Steps:**
1. Send the same prompt 10 times. Record latency and output each time → notice variance and non-determinism.
2. Note tokens/second from `--verbose`. Double the prompt length → measure the latency change.
3. Add a response cache (hash of prompt → Redis) → repeat requests become instant.
4. Write the cost formula for a hosted model: `requests/day × (input tokens × input price + output tokens × output price)`. Estimate monthly cost for 10k users.
5. Make 10 test questions with expected answers (a mini eval). Change the prompt → re-run → did quality drop?

**Aha:** an LLM is a slow, expensive, non-deterministic dependency. Design around it with timeouts, caching, budgets, and evals.
**Remember it as:** *"Treat the model like a flaky, pricey third-party API."*

---

## Drill tracker

| Drill | Day 0 | Day 1 | Day 7 | Day 30 speed-run | Day 90 twist | My "aha" in one line |
|---|---|---|---|---|---|---|
| D1 p99 | | | | | | |
| D2 stateless | | | | | | |
| D3 consistent hashing | | | | | | |
| D4 caching | | | | | | |
| D5 estimation | | | | | | |
| D6 data models | | | | | | |
| D7 B-tree index | | | | | | |
| D8 covering index | | | | | | |
| D9 LSM | | | | | | |
| D10 encoding | | | | | | |
| D11 replication lag | | | | | | |
| D12 multi-leader | | | | | | |
| D13 quorum | | | | | | |
| D14 CAP/PACELC | | | | | | |
| D15 sharding | | | | | | |
| D16 lost update | | | | | | |
| D17 write skew | | | | | | |
| D18 MVCC | | | | | | |
| D19 sync vs async | | | | | | |
| D20 REST vs gRPC | | | | | | |
| D21 Kafka | | | | | | |
| D22 idempotency | | | | | | |
| D23 outbox | | | | | | |
| D24 saga | | | | | | |
| D25 CQRS/ES | | | | | | |
| D26 resilience | | | | | | |
| D27 rate limiting | | | | | | |
| D28 BOLA | | | | | | |
| D29 observability | | | | | | |
| D30 LLM component | | | | | | |
