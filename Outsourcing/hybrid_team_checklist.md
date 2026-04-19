# Hybrid Team Setup — Implementation Checklist

Based on `hybrid_team_setup.md`. Work through these phases in order — each phase is a prerequisite for the next.

---

## Phase 1 — Vault repo

The foundation. Do this first, before touching any project repo.

- [ ] Create a private GitHub repo named `vault`
- [ ] Clone the repo locally to a folder Obsidian can open
- [ ] **Obsidian setup**
  - [ ] Open the cloned vault folder as an Obsidian vault
  - [ ] Install the **Obsidian Git** plugin — handles commit/push/pull from inside Obsidian
  - [ ] Configure Obsidian Git: auto-pull on startup, auto-commit on save (or on a timer)
  - [ ] Confirm a test note commits and pushes to GitHub correctly
  - [ ] Optional: install **Dataview** plugin for querying notes (useful for inbox triage and strategy review)
  - [ ] Optional: install **Templater** plugin to standardize daily note and inbox entry format
- [ ] Write `CLAUDE.md` — how the vault is organized, conventions, reading order for agents
- [ ] Create folder structure:
  - [ ] `daily/` — daily notes, chronological
  - [ ] `strategy/` — product thinking, design exploration, architectural ideas, cross-project patterns (rough, pre-crystallised)
  - [ ] `decisions/` — personal record of *why* you made each decision (narrative, not agent-readable constraints)
  - [ ] `skills/` — custom Claude Code skills
  - [ ] `inbox/` — pressure valve, sorted weekly
- [ ] Write a seed `strategy/` note for each active project (even one paragraph is enough to start)
- [ ] Write a seed `decisions/` entry for one non-obvious call you've already made
- [ ] Note: vault docs are for **your thinking** — when a decision is ready to constrain agent behaviour, graduate it to the project repo (see Phase 2)
- [ ] Commit and push

---

## Phase 2 — Project repo(s)

Repeat for each active project. One repo per product.

These files are **agent-readable constraints** — an agent reading only this repo must have enough context to implement an issue correctly. Keep them distilled and current.

- [ ] `CLAUDE.md` — how to work in this codebase, conventions, test expectations (entry point for every agent session)
- [ ] `ARCHITECTURE.md` — why the system is shaped this way; agents read this before making structural choices
- [ ] `DECISIONS.md` — ADR-style log of non-obvious choices as constraints ("do X, not Y, because Z"); graduate from `vault/decisions/` when a decision must constrain agent behaviour
- [ ] `PRODUCT.md` — current priorities, roadmap, the why; agents use this to draft and rank issues
- [ ] `DESIGN.md` — design constraints agents must follow when building UI or components (patterns, naming conventions, component structure); **not** design exploration — that lives in `vault/strategy/`
- [ ] `skills/` folder — project-specific Claude Code skills (can be empty to start)
- [ ] Confirm all files are committed and up to date before any agent session touches the repo

**Graduation trigger:** if you make a decision in `vault/decisions/` that would cause an agent to build the wrong thing if it didn't know about it — it's time to add it to this repo's `DECISIONS.md`.

---

## Phase 3 — GitHub configuration

- [ ] Enable **GitHub Projects v2** — create one board spanning all active project repos
- [ ] Create board columns: `Backlog` · `In Progress` · `In Review` · `Done` · `Future`
- [ ] Create type labels: `bug` · `feature` · `chore` · `future`
- [ ] Create `needs-type` label — applied automatically when a type label is missing
- [ ] Create **4 issue templates** (one per type), each with the right brief structure:
  - [ ] `bug.md` — repro steps · severity · expected vs actual behaviour · acceptance criteria
  - [ ] `feature.md` — user value · scope · what NOT to do · acceptance criteria · files likely involved
  - [ ] `chore.md` — what · why now · definition of done
  - [ ] `future.md` — idea + rationale (lightweight — this won't be actioned immediately)
- [ ] Set up **GitHub Action: universal board membership** (`issues.opened` trigger):
  - [ ] Add issue to Projects v2 if not already present
  - [ ] Check for type label — add `needs-type` if missing
  - [ ] Test by manually opening an issue and confirming it lands on the board automatically
- [ ] Set up **GitHub Actions** for CI on each project repo (tests, lint, whatever fits)
- [ ] Optional: add a doc-freshness gate (Action that reminds you to update `DECISIONS.md` on merge to main)

---

## Phase 4 — Skills

- [ ] **Weekly synthesis skill** (highest priority — this closes the loop)
  - [ ] Define what it reads: week's `daily/` notes + merged PRs
  - [ ] Define what it writes: a synthesis note into `vault/strategy/`
  - [ ] Write the skill in `vault/skills/weekly_synthesis.md`
  - [ ] Run it once manually to validate output before scheduling
- [ ] **Backlog health monitor + issue drafting skill**
  - [ ] Decide your threshold X — maximum open issues without `future` label (start with 5–10)
  - [ ] Create the `future` label in GitHub — this is the escape valve for parked work
  - [ ] Define trigger conditions: after every merge + daily schedule
  - [ ] Define what the monitor reads: GitHub Issues API (count open, filter out `future` label)
  - [ ] Define candidate search sources: `vault/strategy/` · `vault/inbox/` · `vault/decisions/` · `project/PRODUCT.md`
  - [ ] Define the brief structure: context, what to build, acceptance criteria, what NOT to do, files likely involved, suggested priority
  - [ ] Define the three confirmation outcomes: approve as-is → create issue, edit then approve → create issue, discard → back to strategy/
  - [ ] Define what "add to board" means: issue created, Projects v2 updated, labels + milestone set
  - [ ] Write `vault/skills/backlog_monitor.md` — monitoring + candidate search + brief generation
  - [ ] Write `vault/skills/issue_draft.md` — type-aware brief drafting (called by monitor, also usable standalone):
    - [ ] Skill classifies candidate as bug · feature · chore · future before drafting
    - [ ] Skill applies the correct template per type
    - [ ] Skill sets the type label on creation
  - [ ] Test on a board with known count < X and verify it drafts the right number and correct types
  - [ ] Set up GitHub Action or scheduled task to run `backlog_monitor.md` daily
- [ ] **Meeting capture skill** — extracts key decisions, actions, signals from meeting notes into `inbox/`
- [ ] **Inbox sort skill** — triages `inbox/` entries into `daily/`, `strategy/`, or triggers issue drafting skill
- [ ] Add skills to `vault/CLAUDE.md` with usage notes

---

## Phase 5 — Merge-time discipline

The two moments most likely to slip. Build the habit before it's needed.

- [ ] Write a personal merge checklist (can live in `vault/decisions/merge_checklist.md`):
  - [ ] Does this change a non-obvious architectural choice? → update `ARCHITECTURE.md`
  - [ ] Did I make a decision I'd want future-me (or an agent) to understand? → update `DECISIONS.md`
  - [ ] Does the README or CHANGELOG need a line? → add it while context is warm
- [ ] Run through the checklist on your next 3 merges to build the muscle memory

---

## Phase 6 — Weekly rhythm

- [ ] Schedule a fixed weekly slot (e.g., Friday EOD or Monday morning) for:
  - [ ] Run the weekly synthesis skill
  - [ ] Sort `inbox/` — nothing carries over unsorted for more than a week
  - [ ] Review `strategy/` notes — promote any vault insight that has crystallized into a project `DECISIONS.md`
  - [ ] Groom the Projects v2 board — close stale issues, write new ones
- [ ] After 4 weeks: review whether the synthesis notes are actually useful. If not, adjust what the skill reads or writes.

---

## Health checks (run monthly)

- [ ] Is every project repo's `CLAUDE.md` still accurate? Agents read this — stale context causes bad output.
- [ ] Is `vault/CLAUDE.md` still an accurate map of the vault? Update if structure has drifted.
- [ ] Are there vault `strategy/` notes older than 60 days that haven't influenced any project decision? Either act on them or archive them.
- [ ] Are any `decisions/` entries outdated or reversed? Mark them as superseded rather than deleting.

---

## Re-evaluation triggers (from the doc)

Revisit the whole setup if any of these become true:

- A human teammate joins who won't touch Git
- External stakeholders need polished roadmap sharing monthly+
- A cycle cadence emerges with genuine participants beyond yourself
