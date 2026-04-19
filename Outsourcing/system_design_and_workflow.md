# System Design & Workflow

> **Diagrams**
> - [System flow & HITL checkpoints](./diagram_system_flow.mermaid)
> - [Automation model](./diagram_automation.mermaid)

Operating model for a solo human working alongside Claude Code agents across multiple projects. This document describes how the system is structured, how work flows through it, and how to interact with agents effectively.

---

## 1. System design

### The invariant

Everything is a Git repo with a `CLAUDE.md`. Context lives next to the work it describes. Agents read shared repos — they do not carry their own memory between sessions. This means the quality of what agents produce is directly proportional to the quality of what you've written down.

### The two repo families

```
vault/                        ← personal brain, cross-project
├── CLAUDE.md                 ← how the vault works, reading order for agents
├── daily/                    ← daily notes, chronological
├── strategy/                 ← product thinking, design exploration, cross-project patterns
├── decisions/                ← personal record of decisions and reversals (the "why" narrative)
├── skills/                   ← custom Claude Code skills
└── inbox/                    ← pressure valve, sorted weekly

project-repo/                 ← one per active product
├── CLAUDE.md                 ← how to work in this codebase, conventions, test expectations
├── ARCHITECTURE.md           ← why the system is shaped this way
├── DECISIONS.md              ← ADR-style log of decisions as agent-readable constraints
├── PRODUCT.md                ← current priorities, roadmap, the why
├── DESIGN.md                 ← design constraints agents must follow (patterns, conventions, naming)
├── skills/                   ← project-specific Claude Code skills
└── src/                      ← the code
```

**Vault** is your brain. It holds cross-project thinking, personal decisions, and the skills that automate recurring tasks. It's private. Agents read it at the start of vault sessions.

**Project repos** are execution contexts. Each repo is self-contained: an agent reading only that repo should have enough context to implement an issue correctly. They are hosted on GitHub alongside the vault.

### Where docs live — the decision rule

The test for any document: **does an agent need this to do its job in this codebase?**

| If yes → project repo | If no → vault |
|----------------------|---------------|
| `DECISIONS.md` — constraints agents must not violate | `vault/decisions/` — the personal narrative of *why* you decided something |
| `ARCHITECTURE.md` — system shape agents must respect | `vault/strategy/` — architectural thinking before it crystallises |
| `PRODUCT.md` — priorities agents use to draft and rank issues | `vault/strategy/` — product direction, half-formed ideas, cross-project patterns |
| `DESIGN.md` — design constraints agents must follow when building UI or components | `vault/strategy/` — design exploration, visual thinking, early-stage research |

**The graduation path — thinking becomes constraint:**

```
vault/strategy/        ← you think and explore here (rough, reversible, cross-project)
      ↓ crystallises
vault/decisions/       ← you record the decision and your reasoning (personal record)
      ↓ ready to constrain agent behaviour
project/DECISIONS.md   ← distilled constraint, agent-readable (no reasoning noise, just the rule)
```

A decision that lives only in `vault/decisions/` is invisible to agents working in the project repo — which is fine until it affects what they should build. That's the signal to graduate it.

**What stays in the vault permanently:** cross-project patterns, decisions that apply to how *you* work rather than how the codebase works, design explorations that never shipped, strategic thinking that informed a direction but doesn't need to constrain implementation.

### GitHub as the execution layer

GitHub connects the two families and provides:

- **Issues** — self-contained briefs, agent-readable, linked to Projects v2
- **Projects v2** — roadmap view across all active repos
- **Pull requests** — agents open them, you review them
- **Actions** — CI, lint, doc-freshness gates
- **Merge** — the moment to update `DECISIONS.md` and `ARCHITECTURE.md` while context is warm

### Obsidian as the vault interface

The vault is a plain Git repo. Obsidian opens it as a local folder. The **Obsidian Git** plugin handles commit/push/pull automatically — every note you write in Obsidian becomes a committed markdown file on GitHub. This means agents, Obsidian, and GitHub are always reading the same files.

---

## 2. Day-to-day workflow

### Morning

1. Pull latest vault and any active project repos
2. Open `daily/YYYY-MM-DD.md` — note the day's focus, carry over anything unresolved from yesterday's note
3. Check GitHub Projects v2 — review open issues, pick the day's work
4. If an issue needs more context before an agent session, sharpen the brief first (see Section 4)

### During the day — capturing signals

Anything worth keeping goes into `inbox/` or directly into `daily/`. The rule: if you'd want to act on it or think about it later, it goes in. Don't filter too hard at capture time — that's what the weekly sort is for.

Signals worth capturing:
- Customer feedback or user complaints
- A decision you made and why
- A market or competitor observation
- A blocker or dependency you encountered
- An idea that arrived mid-task

### Running an agent session

1. Open Claude Code in the target project repo
2. The agent reads `CLAUDE.md` automatically — no need to re-explain conventions
3. Point the agent at the issue: `implement issue #N`
4. Agent reads the issue brief, reads relevant files, implements, opens a PR
5. You review the PR (agent can assist with checks — see Section 4)
6. On merge: update `DECISIONS.md` and `ARCHITECTURE.md` if the change warrants it

### End of day

- Write any unresolved thoughts into `daily/` or `inbox/`
- If a decision crystallized today, add it to `decisions/` while context is warm
- Push vault

---

## 3. Agent interaction model

### What agents read

Agents do not carry memory between sessions. Every session starts cold. What they know is what you've written down. Before each session, the agent reads:

- The repo's `CLAUDE.md` (always — this is the entry point)
- The issue brief (for implementation sessions)
- Any files referenced in the issue
- `DECISIONS.md` and `ARCHITECTURE.md` when making structural choices

The vault is read in vault sessions (synthesis, inbox sort, research). It is not read in project repo sessions unless you explicitly point the agent at it.

### Two modes

**Primary work (agent does the thing)**
- Implementing a GitHub issue into a PR
- Running the weekly synthesis skill
- Sorting the inbox
- Extracting notes from a meeting recording

**Assistance (agent helps you do the thing)**
- Reviewing a PR for correctness, security, edge cases
- Drafting a strategy note from rough bullet points
- Proposing `DECISIONS.md` or `ARCHITECTURE.md` updates after a merge
- Researching a technical question against the current codebase

### The discipline

Agents execute. You decide. Specifically:

- You decide what goes into an issue brief
- You decide what matters enough to land in `strategy/` or `decisions/`
- You review every PR before merge
- You decide when a vault insight is ready to graduate into a project decision

Agents capture, implement, synthesize drafts, and run checks. They do not set priorities, they do not decide what's worth keeping, and they do not merge their own PRs.

---

## 4. Issue drafting & product board population

Issue brief quality is the single highest-leverage act in the system. Rather than writing briefs from scratch, AI drafts them and you review. The leverage shifts from writing to reviewing — which is faster and still keeps you in control of what gets built.

### Backlog health rule

AI maintains a maximum of **X open issues** that do not carry the `future` label (X is your configured threshold — start with 5–10). The monitor runs automatically after every merge and on a daily schedule.

```
open issues (without 'future' label) < X  →  AI searches for candidates → drafts briefs → you confirm → AI creates + adds to board
open issues (without 'future' label) ≥ X  →  no action, board is healthy
```

The `future` label is the escape valve — any issue you want to park without it counting toward the active limit gets that label. This keeps the active board lean and the drafting loop focused on near-term work.

### The flow

```
After merge / on schedule
        ↓
🤖 Monitor counts open issues without 'future' label
        ↓ (if count < X)
🤖 Searches vault/strategy/ · inbox/ · PRODUCT.md for candidates
        ↓
🤖 Drafts issue brief per candidate
        ↓
👤 You review & confirm (edit / approve / discard)    ⚑ HITL checkpoint 4
        ↓ (on approval)
🤖 Creates issue + adds to Projects v2
        ↓
🤖 Re-checks count → loops until count = X
```

**Step 1 — Trigger a draft**
Runs automatically after merge or on schedule. Can also be triggered manually:
```
claude run vault/skills/backlog_monitor.md
```

**Step 2 — AI checks board health**
Counts open GitHub issues that do not have the `future` label. If count ≥ X, exits. If count < X, proceeds to candidate search.

**Step 3 — AI searches for candidates**
Reads `vault/strategy/`, `vault/inbox/`, `vault/decisions/`, and `project/PRODUCT.md`. Ranks candidates by recency and strategic fit. Drafts one brief per candidate.

**Step 4 — AI reads context and drafts**
The skill reads `vault/strategy/`, `project/DECISIONS.md`, and `project/PRODUCT.md` to produce a structured brief:

```markdown
## Context
What is this? Why does it exist? What problem does it solve?
Link to relevant strategy/ or decisions/ notes.

## What to build
Specific, unambiguous description of what the agent should produce.
File paths, function names, API shapes where relevant.

## Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## What not to do
Approaches that seem obvious but contradict DECISIONS.md.

## Files likely involved
- `src/path/to/file.ts`
- `ARCHITECTURE.md` (read before making structural choices)

## Suggested priority / milestone
[Derived from PRODUCT.md]
```

**Step 3 — You review and confirm (HITL checkpoint 4)**
Three outcomes:
- **Approve as-is** → AI creates the GitHub issue and adds it to Projects v2
- **Edit then approve** → you amend the draft, then AI creates and adds to board
- **Discard** → brief goes back to `strategy/` as a note; nothing is created

**Step 4 — AI populates the board**
On confirmation, the skill automatically:
- Opens the GitHub issue with the confirmed brief
- Adds it to Projects v2
- Sets priority, milestone, and labels based on `PRODUCT.md`
- Links to any parent epic if one exists

### Issue types & board membership

Every issue — regardless of how it was created — must be on the Projects v2 board and must carry exactly one type label. A GitHub Action enforces this on `issues.opened`.

| Type | Label | Board column | Counts toward X | Brief template |
|------|-------|-------------|----------------|----------------|
| Bug | `bug` | Backlog (top of queue) | ✅ Yes | Repro steps · severity · expected vs actual |
| Feature | `feature` | Backlog | ✅ Yes | User value · scope · acceptance criteria |
| Chore / tech debt | `chore` | Backlog | ✅ Yes | What · why now · definition of done |
| Parked / future | `future` | Future column | ❌ No | Idea + rationale only |

The `future` label is the **only exemption** from the active count. Everything else — bugs filed mid-session, features drafted by AI, chores added manually — counts toward X and sits in the active backlog.

**What the GitHub Action does on `issues.opened`:**
1. Checks whether the issue is already on the Projects v2 board — adds it if not
2. Checks whether a type label (`bug`, `feature`, `chore`, `future`) is present — adds `needs-type` if missing
3. Runs for every issue, regardless of who or what created it

This means a bug you file manually at 2am during an incident lands on the board automatically, with no extra steps.

### Signs a brief needs more work before you approve

- The context section doesn't explain *why*, only *what*
- Acceptance criteria are ambiguous or untestable
- "What not to do" is empty — almost always a gap
- The brief doesn't reference any `DECISIONS.md` entries, but the feature touches an area with prior decisions

When any of these are true, edit the draft rather than approving and correcting the agent mid-session. A five-minute edit before approval saves a full revision cycle after the PR.

---

## 5. Vault ↔ project repo flow

The vault and project repos are intentionally separate. Translation between them is a deliberate human act, not an automatic sync. This prevents noise from flowing into execution context.

### The graduation path

```
External signal
    ↓
vault/inbox/          ← raw capture, unsorted
    ↓ (weekly sort)
vault/daily/          ← if it's context for today
vault/strategy/       ← if it needs thinking
    ↓ (when crystallized)
vault/decisions/      ← your personal record of the decision
    ↓ (when ready to act)
project/DECISIONS.md  ← the decision as a constraint for agents
project issue brief   ← the decision as a piece of work
```

### When to graduate a vault note

A strategy note is ready to graduate when:
- You've stopped second-guessing the direction
- The decision has implications for how agents should behave in the codebase
- You're about to open a GitHub issue that depends on it

A vault note that never graduates is fine — not everything needs to become code. But a note that *should* graduate and doesn't will silently corrupt agent output: the agent will make choices that contradict thinking you've done but never wrote into the repo.

### Keeping the translation intentional

Do not auto-sync vault notes into project repos. The vault contains rough thinking, reversals, dead ends, and half-formed ideas. Project repos should contain only crystallized decisions. The human review step — reading a strategy note and deciding it's ready to become a `DECISIONS.md` entry — is what keeps the project context clean.

---

## 6. Weekly rhythm

The weekly synthesis loop is what prevents the vault from becoming a historical record instead of a living brain.

### Friday EOD (or Monday morning — pick one and protect it)

**Step 1 — Run weekly synthesis skill**
```
claude run vault/skills/weekly_synthesis.md
```
The skill reads:
- `daily/` notes from the past 7 days
- Merged PRs across all active project repos
- Any new `inbox/` entries

It produces a synthesis note in `vault/strategy/YYYY-WW.md` summarizing:
- What shipped
- What decisions were made
- What signals are worth carrying forward
- Open questions going into next week

**Step 2 — Sort inbox**
Go through every file in `vault/inbox/`. For each:
- Discard: delete it
- Context: move to `daily/` with a date note
- Thinking needed: move to `strategy/` as a new or appended note
- Ready to act: open a GitHub issue, then delete the inbox entry

Nothing carries over in `inbox/` for more than a week. An unsorted inbox is a guilt pile, not a pressure valve.

**Step 3 — Review strategy notes**
Scan `vault/strategy/`. For each note older than 2 weeks:
- Is this still live thinking? Leave it.
- Has it crystallized into a decision? Graduate it to `decisions/` and the relevant project repo.
- Is it stale or superseded? Archive or delete it.

**Step 4 — Groom the board**
In GitHub Projects v2:
- Close issues that are no longer relevant
- Update priorities based on the week's synthesis note
- Write any new issues that emerged from the inbox sort
- Confirm the top 3 issues for next week are well-briefed

**Step 5 — Push vault**
Commit and push everything. The week's work is now readable by agents in the next session.

---

## 7. End-to-end flow of one piece of work

```
1. Signal arrives (user feedback, market observation, idea)
         ↓
2. Captured into vault/inbox/ or vault/daily/
         ↓
3. Matures in vault/strategy/ through thinking and weekly synthesis
         ↓
4. Crystallizes → added to vault/decisions/ + project/DECISIONS.md
         ↓
5. 🤖 AI drafts issue brief from strategy note + DECISIONS.md + PRODUCT.md
         ↓
6. 👤 You review draft — edit, approve, or discard       ⚑ HITL checkpoint 4
         ↓
7. 🤖 AI creates GitHub Issue + adds to Projects v2 (on approval)
         ↓
8. 🤖 Agent session: reads CLAUDE.md + issue + relevant files → implements → opens PR
         ↓
9. 👤 You review PR (agent assists with checks)           ⚑ HITL checkpoint 5
         ↓
10. Merge → update DECISIONS.md + ARCHITECTURE.md while context is warm
         ↓
11. Weekly synthesis reads the merge → updates vault/strategy/ → loop closed
```

Each step is a gate. Work that skips a step — especially steps 3–7 — produces PRs that need heavy revision or contradict decisions already made. The brief review (step 6) is where the leverage sits: a five-minute edit before approval saves a full revision cycle after the PR.

---

## 8. Where leverage compounds

Rank-ordered interventions by payback:

1. **`vault/CLAUDE.md` + `vault/skills/`** — read every vault session, enables automation, compounds across all projects
2. **Weekly synthesis skill** — closes the loop from execution to strategy; without it the vault decays
3. **Issue brief quality** — determines agent output quality; the single highest-leverage PM act
4. **`DECISIONS.md` discipline at merge time** — prevents agents from undoing deliberate choices months later
5. **External signal filter** — "will I or an agent benefit from this in six months?" keeps the vault dense and useful

Everything else is plumbing.
