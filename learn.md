# learn.md — Learning Log

Theory source: **Ahmed Elemam – أحمد الإمام** YouTube channel
(playlists: *System Design بالعربي*, *Designing Data Intensive Applications بالعربي*,
*DataBase Deep Dive*, *Microservices بالعربي*, *Security بالعربي*,
*OBSERVABILITY FROM SCRATCH*, *ما وراء النقاشة – Advanced Frontend*).

## Status legend

`⬜` not started · `👀` watched + notebook notes · `📝` trade-off box written · `🧪` lab done · `🔁` recalled 3 times

A video only reaches 🧪 when its linked lab in `practice.md` has a `RESULTS.md`.

---

## Notes live in my paper notebook

I take notes **by hand, with diagrams**, in my own words. This file only tracks status.
At the end of every video's notebook page, I draw this box:

```
┌─────────────────────────────────────────────────┐
│ #NN  <short title>              Date: ____       │
│ Trade-off: ___ gives me ___ but costs me ___     │
│ When I would NOT use this: ___                   │
│ Recall: 1 day [ ]  1 week [ ]  1 month [ ]       │
└─────────────────────────────────────────────────┘
```

Claude can't see the notebook, so after each video it asks me 2–3 short questions
to answer from memory. (Optional: upload a photo of a page for feedback.)

---

## Roadmap checklist

### Level 1 — Basics (System Design بالعربي)
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| 1 | Scale your application from zero to millions of users – part 1 | 7m | Lab 1 | 👀 |
| 2 | Scale your application … – part 2 | 5m | Lab 1 | 👀 |
| 3 | Scale your application … – part 3 | 6m | Lab 1 | ⬜ |
| 4 | Scale your application … – part 4 | 3m | Lab 1 | ⬜ |
| 5 | System Design fundamentals – تيك بودكاست | 1h09 | Lab 0 | ⬜ |

### Level 2 — How data is stored (DDIA ch. 1–4)
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| 6 | شرح كتاب Designing Data Intensive Applications – الفصل الأول | 31m | Lab 0 | ⬜ |
| 7 | SQL vs NoSQL – Chapter 2.1 | 14m | Lab 2 | ⬜ |
| 8 | MapReduce & Graph database – Chapter 2.2 | 22m | Lab 2 | ⬜ |
| 9 | How Database Indexes really works? – Chapter 3.1 | 30m | Lab 3 | ⬜ |
| 10 | Database Indexes B-trees, B+trees vs LSM tree – Chapter 3.2 | 15m | Lab 3 | ⬜ |
| 11 | Clustered, covering, multi-column & fuzzy index – Ch 3.3 | 30m | Lab 3 | ⬜ |
| 12 | السر وراء سرعة ال NoSQL Databases | 20m | Lab 3 | ⬜ |
| 13 | Data Encoding: Apache Thrift & Protocol Buffers – Ch 4.1 | 18m | Lab 4 | ⬜ |
| 14 | AVRO vs Thrift vs Protocol Buffers – Ch 4.2 | 16m | Lab 4 | ⬜ |

### Level 3 — Distributed data (DDIA ch. 5–6)
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| 15 | Data Replication – Single Leader Replication – CH5 | 24m | Lab 5 | ⬜ |
| 16 | Eventual Consistency – Chapter 5.2 | 10m | Lab 5 | ⬜ |
| 17 | Multi Leader Replication – Chapter 5.3 | 11m | Lab 5 | ⬜ |
| 18 | Dynamo – Leaderless & Quorum Replication – Ch 5.4 | 15m | Lab 5 | ⬜ |
| 19 | What is Data Consistency – Chapter 5.5 | 9m | Lab 5 | ⬜ |
| 20 | Database Sharding – Partitioning – Ch 6 | 22m | Lab 6 | ⬜ |
| 21 | MongoDB Sharding & Replication production-like demo – Ch 6.2 | 38m | Lab 6 | ⬜ |

### Level 4 — Database deep dives
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| 22 | PostgreSQL Deep Dive with Hussein Nasser | 2h04 | Lab 7 | ⬜ |
| 23 | MongoDB Deep Dive with Hussein Nasser | 2h01 | Lab 7 | ⬜ |

### Level 5 — Real-world architecture & microservices
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| 24 | Real world system design with Bassem Dghaidi | 1h51 | Lab 8 | ⬜ |
| 25 | From Monolith to Microservices with Alaa Attya | 1h30 | Lab 8 | ⬜ |
| 26 | MicroServices Architecture with Mohamed Sweelam | 2h01 | Lab 8 | ⬜ |
| 27 | MicroServices Communication Patterns (Sync/Async/REST/gRPC/Protobuf) | 1h00 | Lab 8 | ⬜ |
| 28 | Kafka with Mohamed Ragab | 2h09 | Lab 9 | ⬜ |
| 29 | Kafka Transaction with Mohamed Ragab | 1h02 | Lab 9 | ⬜ |
| 30 | Distributed Transaction with Mostafa Mansour | 1h00 | Lab 9 | ⬜ |
| 31 | CQRS with Mohammed Yahia | 1h31 | Lab 9 | ⬜ |
| 32 | Real-world System Design with Ahmed Farghal – Restate durable execution | 1h59 | Lab 10 | ⬜ |

### Level 6 — Production
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| 33 | API Security with Mohammed El Sherif | 1h00 | Lab 11 | ⬜ |
| 34 | Secure Systems Development Lifecycle (SSDLC) for Startups | 59m | Lab 11 | ⬜ |
| 35 | OBSERVABILITY FROM SCRATCH – Part 1 | 1h29 | Lab 12 | ⬜ |
| 36 | OBSERVABILITY FROM SCRATCH – Part 2 | 44m | Lab 12 | ⬜ |

### Level 7 — Interview practice
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| 37 | System Design Interview with Ahmed Soliman | 2h05 | Lab 13 | ⬜ |
| 38 | Tracking Stock Prices – System Design Interview | 1h23 | Lab 13 | ⬜ |
| 39 | Design YouTube PayOut System – System Design Interview | 1h55 | Lab 13 | ⬜ |
| 40 | Low Level Design a vending machine | 1h32 | Lab 13 | ⬜ |

### Optional — Data engineering
| Video | Lab | Status |
|---|---|---|
| Batch & Stream processing with Ahmed Elsayed | Capstone | ⬜ |
| Building Data Pipeline – تطبيق عملي | Capstone | ⬜ |
| Data Solution Architecture with Moustafa Mahmoud – Part 1 | — | ⬜ |
| Data Aggregation using Columnar Database ClickHouse with Sayed Alesawy | — | ⬜ |

### Frontend track (ما وراء النقاشة – Advanced Frontend)
| # | Video | Length | Lab | Status |
|---|---|---|---|---|
| F1 | Frontend Roadmap | 2h10 | — | ⬜ |
| F2 | Overview on Frontend Frameworks with Ahmad Alfy | 2h07 | F2 | ⬜ |
| F3 | From javascript to typescript with Ahmed Hassanein | 1h55 | F4 | ⬜ |
| F4 | React vs Vue vs Angular vs Qwik vs Svelte with Abdelrahman Awad | 2h29 | F2 | ⬜ |
| F5 | How to enhance front-end app performance with Medhat Dawoud | 2h22 | F1 | ⬜ |
| F6 | GraphQL from Zero to Zero with Abdelrahman Awad | 1h15 | F3 | ⬜ |
| F7 | Web Accessibility (a11y) with عبدالعزيز الشمّاسي | 2h06 | F5 | ⬜ |
| F8 | Building JS Library with Abdelrahman Awad | 1h18 | — | ⬜ |

### AI-native track (after Level 5)
| Playlist / video on the channel | Lab | Status |
|---|---|---|
| Building AI Agents (playlist) | Capstone | ⬜ |
| Build your own MCP server | Capstone | ⬜ |
| How to be 10X SW Engineer using MCP? | — | ⬜ |
| دليلك كمبرمج لتشغيل ال LLMs علي جهازك الشخصي | Capstone | ⬜ |

---

## Concepts I must explain in 60 seconds (recall bank)

Tick a concept only when you can explain it out loud, with an example, without notes.
Each concept has a hands-on drill in `drills.md` (D1–D30). Do the drill, then come back and tick it.

- [ ] Latency vs throughput; why p99 matters more than average
- [ ] Vertical vs horizontal scaling; stateless services
- [ ] Load balancing algorithms (round robin, least connections, consistent hashing)
- [ ] Cache-aside vs write-through vs write-back; cache invalidation & stampede
- [ ] SQL vs document vs graph models — when each wins
- [ ] B-tree vs LSM-tree: read/write/space amplification
- [ ] Clustered vs secondary vs covering vs composite index (and column order)
- [ ] Schema evolution: forward vs backward compatibility
- [ ] Single-leader, multi-leader, leaderless replication
- [ ] Replication lag problems: read-your-writes, monotonic reads
- [ ] Quorums: why W + R > N
- [ ] Sharding: range vs hash keys, hot partitions, rebalancing
- [ ] Isolation levels: dirty read, lost update, write skew, phantoms
- [ ] MVCC and why Postgres needs VACUUM
- [ ] CAP and PACELC in plain words
- [ ] Sync vs async communication; REST vs gRPC
- [ ] Kafka partitions, consumer groups, ordering guarantees
- [ ] Idempotency keys; at-least-once vs exactly-once (and why "exactly-once" has fine print)
- [ ] Outbox pattern; Saga (choreography vs orchestration)
- [ ] CQRS and event sourcing — and when NOT to use them
- [ ] Timeouts, retries with backoff + jitter, circuit breakers
- [ ] Rate limiting: token bucket vs sliding window
- [ ] OWASP API risks: broken object-level authorization (BOLA)
- [ ] Logs vs metrics vs traces; RED and USE methods; SLI/SLO/error budget
- [ ] Back-of-the-envelope estimation (QPS, storage, bandwidth)
- [ ] LLM system constraints: token cost, latency, rate limits, caching, evals

---

## My notes

In my paper notebook (see "Notes live in my paper notebook" above).