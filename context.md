# context.md — My Learning Memory (upload at the start of every chat)

> I use Claude in **Incognito mode**, so Claude has no memory between chats.
> This file IS the memory. Keep it short (under ~2 pages). Update it at the end of every session.

---

## 🔁 Ritual

**Start of a new chat** — upload `context.md` (+ only the kit file needed today, e.g. `drills.md`), then paste:

```
This is my context.md. Read it, follow "Rules for Claude", tell me in 3 lines
where we stopped, then continue from "Next step".
```

**End of a session (or any time files need updating)** — type:

```
/sync-chat-with-md
```

Then download the files and **replace** the old ones (keep a copy in your Git repo, so you also get history for free with `git log`).

---

## ⌨️ My commands
When I type a command, follow its definition exactly. No extra explanation needed.

**`/sync-chat-with-md`** — sync this chat into my kit files:
1. Review everything decided, done, or measured in this chat.
2. Always update `context.md`: Current state, Progress, Re-drills due, a 3–5 line Session log entry, and Next step.
3. Update any other kit file **only** if this chat changed it (a rule, a status in `learn.md`, results, etc.). Use the latest version of each file: the one I uploaded, or else the repo on GitHub.
4. Give me the **full updated files** to download (not snippets), with one line per file saying what changed, and list which files stayed unchanged.
5. Never invent results: only record what actually happened in the chat.

## 👤 About me
- Goal: become strong in system design (backend first, frontend track later), with hands-on experience, not only theory.
- Theory source: Ahmed Elemam's YouTube channel (Arabic). Ordered list in `learn.md`.
- Repo: https://github.com/hossam2000ayman/system-design-lab (push before each new chat so it's up to date)
- Language for replies: English.
- Time budget: ~6–8 h/week (see README weekly rhythm).
- My OS / setup: _(fill in: Mac / Windows / Linux, Docker installed? k6 installed?)_

## 📁 My kit files
`README.md` (guide + AI rules) · `learn.md` (40 videos + status; notes are in my paper notebook) · `drills.md` (D1–D30 concept drills + re-drill schedule) · `practice.md` (Labs 0–13 + capstone + frontend) · `CLAUDE.md` (rules for coding agents) · `context.md` (this file)

## 🤖 Rules for Claude (when reading this file)
1. Coach me step by step; one step at a time, then wait for my answer.
2. Never give me the answer before I write my own prediction or design.
3. After I share results, ask me *why* before explaining.
4. Keep the "Re-drill schedule" in mind: remind me which old drills are due (Day 1 / 7 / 30 / 90).
5. Keep updates to this file short; summarize, don't copy the chat.
6. My notes are handwritten (paper notebook + diagrams), not in md files. You can't see them, so after each video ask me 2–3 short questions to answer from memory, and check that I wrote the trade-off + "when NOT" box.

---

## 📍 Current state
- **Session:** 1 — first study session (Day 0), Oct 3, 2026
- **Working on:** Drill **D1 (latency vs p99)**
- **Stopped at:**
  - [x] Step 1 — watched videos #1–2, notes in paper notebook (status 👀 in `learn.md`)
  - [ ] Step 1b — add the trade-off + "when NOT" box to the #1 and #2 notebook pages (→ 📝)
  - [ ] Step 2 — write predictions for D1 (avg, p95, p99, req/s with 50 VUs, 30 s; server = 5–50 ms, 1% of requests take 2 s)
  - [ ] Step 3 — check setup: `python3 --version`, `k6 version`
- **Next step:** Send D1 predictions + setup result → Claude guides me to run D1 and read results.

## 🎯 My predictions & results (latest drill)
| Drill | Metric | My prediction | Measured | Why the difference |
|---|---|---|---|---|
| D1 | avg | | | |
| D1 | p95 | | | |
| D1 | p99 | | | |
| D1 | req/s | | | |

## 📈 Progress
- Videos done: 2 / 40 (👀 #1, #2)
- Drills done (Day 0): none
- Labs done: none
- **Re-drills due:** none yet

## 🧠 Key "aha" moments (one line each, my own words)
- _(empty)_

## ❓ Open questions
- _(empty)_

---

## 🗒️ Session log (newest at the bottom, 3–5 lines each)

**Session 0 — Planning (Oct 2, 2026)**
- Found Ahmed Elemam's playlists (via Claude in Chrome) and ordered 40 videos into 7 levels + frontend + AI tracks.
- Built the kit: README, learn, practice, drills, CLAUDE.md.
- Agreed: no checkmark without hands-on; re-drill on Day 1/7/30/90.
- Incognito chats can't be shared or saved → this `context.md` is my memory.

**Session 1 — Day 0: videos #1–2 + D1 (Oct 3, 2026)**
- Watched videos #1–2. Decided no DDIA prerequisite: overview first, depth later (breadth → depth → breadth).
- Rule change: notes are handwritten in a paper notebook with diagrams; `learn.md` tracks status only. Kept the trade-off + "when NOT" box and Claude's verbal check-ins.
- Added the `/sync-chat-with-md` command (defined in "My commands").
- _(rest filled at the end of the session)_