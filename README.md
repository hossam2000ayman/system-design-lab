# System Design Growth Kit

A personal system for going from beginner to advanced in system design, using
Ahmed Elemam's YouTube playlists as the theory source and your own experiments
as the proof that you really understand it.

> **The one rule:** a topic is not "done" when you finish the video.
> It is done when you have **predicted, built, broken, measured, and explained** it.

---

## 1. The files you keep

Put these in one Git repo (for example `system-design-lab`) so your progress is
versioned and visible.

| File | What it is | When you touch it |
|---|---|---|
| `README.md` | This guide: the loop, the rhythm, the AI rules | Read once, re-read monthly |
| `learn.md` | The ordered video roadmap + your notes in your own words | After every video |
| `practice.md` | Hands-on labs mapped to each level, with real-world scenarios | Every week |
| `drills.md` | One small hands-on drill per concept (20–60 min) + a re-drill schedule so you never forget | Day 0, 1, 7, 30, 90 after each concept |
| `CLAUDE.md` | Context and rules for AI coding tools working in this repo (copy it as `AGENTS.md` too) | When you start, then when rules change |
| `decisions/ADR-XXX.md` | Architecture Decision Records: one small file per real decision you made in a lab | Whenever you choose between two options |
| `labs/NN-name/RESULTS.md` | Raw numbers, graphs and conclusions of each lab | End of each lab |

Suggested layout:

```
system-design-lab/
├── README.md
├── learn.md
├── practice.md
├── drills.md
├── CLAUDE.md
├── decisions/
│   └── ADR-001-cache-aside-vs-write-through.md
└── labs/
    ├── 01-scale-from-zero/
    │   ├── docker-compose.yml
    │   ├── loadtest.js
    │   └── RESULTS.md
    └── 05-replication/
        └── ...
```

### ADR template (keep it to half a page)

```markdown
# ADR-001: <decision title>
- Date:
- Context: what problem, what constraints (traffic, consistency, team, cost)
- Options considered: A, B, C
- Decision: B
- Why: the trade-off I accepted
- What would make me change my mind:
- Evidence: link to labs/NN/RESULTS.md
```

---

## 2. The learning loop (one cycle per topic)

```
 ┌──────────┐   ┌──────────┐   ┌─────────┐   ┌─────────┐
 │ 1. LEARN │──▶│2. PREDICT│──▶│ 3. BUILD│──▶│ 4. BREAK│
 └──────────┘   └──────────┘   └─────────┘   └────┬────┘
      ▲                                            │
 ┌────┴─────┐   ┌──────────┐   ┌──────────┐   ┌────▼─────┐
 │8. RECALL │◀──│ 7. TEACH │◀──│6. REFLECT│◀──│5. MEASURE│
 └──────────┘   └──────────┘   └──────────┘   └──────────┘
```

1. **Learn** — Watch the video(s). Write notes in `learn.md` in **your own words**, max one screen per video. No copy-paste, no AI summaries.
2. **Predict** — Before touching code, write a hypothesis *with numbers* in `practice.md`.
   Example: "Adding Redis cache-aside will drop p95 read latency from ~40 ms to under 10 ms, and DB CPU by at least 50%."
3. **Build** — The smallest setup that can test the hypothesis (Docker Compose is enough for almost everything).
4. **Break** — Push load, kill a node, add latency, fill a disk. Real understanding lives in failure modes.
5. **Measure** — p50 / p95 / p99 latency, throughput, error rate, replication lag, CPU/memory. Save the numbers in `RESULTS.md`.
6. **Reflect** — Compare prediction vs. reality. *Why* were you wrong? This step is where most of the learning happens.
7. **Teach** — Explain it in 5 sentences or one diagram (a LinkedIn post, a note to a colleague, or a voice memo). If you can't explain it simply, go back to step 1.
8. **Recall** — Re-answer 3 questions about the topic **without notes** after 1 day, 1 week, and 1 month. Tick the boxes in `learn.md`.
   Even better: **re-do the concept's drill** from `drills.md` (Day 30 = rebuild it from an empty folder, no notes, no AI).

---

## 3. Weekly rhythm (about 6–8 hours/week)

| Day | Time | Activity |
|---|---|---|
| Saturday | 2 h | Learn: watch + notes |
| Sunday | 2 h | Predict + Build |
| Tuesday | 1.5 h | Break + Measure |
| Thursday | 1 h | Reflect + Teach + write ADR |
| Friday | 30 min | Recall quiz + one speed-run drill from `drills.md` on an older topic |

Rough timeline at this pace (adjust freely, consistency beats speed):

| Phase | Levels in `learn.md` | Approx. duration |
|---|---|---|
| Foundations | Levels 1–3 | 5–7 weeks |
| Databases & architecture | Levels 4–5 | 6–8 weeks |
| Production & interviews | Levels 6–7 | 4–6 weeks |
| Capstone (AI-ready system) | `practice.md` → Capstone | 3–4 weeks |

Every 4 weeks: re-read this README, update the status column in `learn.md`, and
write a one-paragraph "what I can do now that I couldn't a month ago".

---

## 4. Becoming a better software engineer in the AI-native SDLC

### What changed

Writing code is now cheap: an agent can produce a service, a migration, or a
Docker Compose file in minutes. What became **scarce and valuable** is:

- **Judgment** — choosing boundaries, data models, consistency levels, and trade-offs.
- **Specification** — describing precisely what should be built and what "correct" means.
- **Verification** — proving with tests, load tests, and observability that it actually works and fails safely.
- **System understanding** — knowing *why* things are slow, inconsistent, or expensive.

That is exactly what system design trains. AI made system design **more**
important, not less: the agent writes the function, you decide the architecture
and you own the consequences.

### Ten principles

1. **Think first, then generate.** When learning, do your own design or prediction *before* asking AI. Then ask it to critique you. Using AI to skip the struggle also skips the learning.
2. **Spec-driven development.** Before prompting an agent, write a short spec: requirements, non-functional requirements (latency, scale, consistency), API contract, data model, and acceptance tests. The spec becomes the thing you review most carefully.
3. **Context engineering.** Agents are only as good as the context you give them. Maintain `CLAUDE.md`/`AGENTS.md`, ADRs, and clear conventions so any tool understands your project.
4. **You own every line you merge.** Read every diff. For each change ask: *how does this fail? what happens under 10x load? what if this call times out?*
5. **Small batches.** Ask for small, reviewable changes. A 2,000-line agent PR is a liability, not productivity.
6. **Tests and evals are your guardrails.** Write or approve tests first; never let an agent weaken a test or a threshold to "make it pass".
7. **Security is still your job.** Check AI-written code for injection, broken authorization, leaked secrets, and **dependencies that don't exist or look suspicious** (verify every new package before installing it).
8. **Learn to design AI systems.** LLM features are distributed systems with new constraints: high and variable latency, token cost, rate limits, non-determinism, caching, retrieval (RAG / vector search), guardrails, evals, and tracing of model calls. Agents and MCP servers are new kinds of services to design and secure. Ahmed's channel has playlists on this: *Building AI Agents*, *Build your own MCP server*, *How to be 10X SW Engineer using MCP*.
9. **Keep fundamentals sharp.** Memory, networking, operating systems, and databases explain the bugs AI can't. See Ahmed's *ما لا يسع المبرمج جهله* playlist (Memory, Computer Architecture, GPU, Serverless).
10. **Writing is the core skill.** Specs, prompts, ADRs, design docs, and PR descriptions are all writing. Clear writing = clear thinking = better agent output.

### How to use AI inside this kit

| ✅ Good use | ❌ Avoid |
|---|---|
| Mock system-design interviewer | Asking for the answer before you've tried |
| Critic of *your* design doc or ADR | Letting AI write your `learn.md` notes |
| Scaffolding Docker Compose / k6 scripts **after** you designed the experiment | Letting the agent choose the architecture |
| Explaining an error or a config option | Pasting results you don't understand |
| Generating recall quizzes from your own notes | Skipping the "Break" and "Measure" steps |

### Prompts you can copy

**Mock interviewer**
```
Act as a senior system design interviewer. Problem: <design X>.
Do not give me the solution. Ask one question at a time, push back on weak
trade-offs, and after 45 minutes score me on: requirements, estimation,
API, data model, scaling, consistency, failure handling, communication.
```

**Design critic**
```
Here is my design doc / ADR. Find the 5 weakest points: missing
requirements, single points of failure, consistency bugs, cost risks,
and what happens at 10x traffic. Rank them by severity. Don't rewrite it.
```

**Recall quiz**
```
Here are my notes on <topic>. Ask me 5 questions, one at a time, that test
understanding, not memorization (trade-offs, failure scenarios, "what if").
Only tell me the correct answer after I respond.
```

**Lab scaffolding**
```
I designed this experiment: <paste hypothesis + setup from practice.md>.
Generate a docker-compose.yml with pinned image versions and a k6 script.
Explain every non-obvious config line in comments. Don't add components
I didn't ask for.
```
