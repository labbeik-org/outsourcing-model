# Skills Database

Comprehensive inventory of all automation skills in the outsourcing model.

## All Skills

```dataview
TABLE
  type as Type,
  trigger as Trigger,
  output as Output,
  hitl as "HITL Required?"
FROM "Outsourcing/skills"
WHERE type
SORT skill-number ASC
```

---

## Skill 1: Meeting Capture 🤖

| Property | Value |
|----------|-------|
| **Type** | Automated (Agent) |
| **Input** | Voice memo, meeting notes |
| **Output** | vault/inbox/ entries |
| **Trigger** | On-demand |
| **HITL** | No |
| **Skill File** | [[meeting_capture]] |

**Purpose:** Extract signals, decisions, and actions from meetings into structured vault entries.

**Status:** 📋 Needs documentation

---

## Skill 2: Inbox Sort 🤖 + 👤

| Property | Value |
|----------|-------|
| **Type** | Hybrid (Agent + Human) |
| **Input** | vault/inbox/ entries |
| **Output** | vault/daily/, vault/strategy/, GitHub Issues, Deleted |
| **Trigger** | Weekly (scheduled) |
| **HITL** | ✅ Checkpoint 1 |
| **Skill File** | [[inbox_sort]] |

**Purpose:** Triage inbox entries and route to appropriate destinations based on human decision.

**Status:** 📋 Needs documentation

---

## Skill 3: Weekly Synthesis 🤖

| Property | Value |
|----------|-------|
| **Type** | Automated (Agent) |
| **Input** | vault/daily/ (past 7 days), Merged PRs |
| **Output** | vault/strategy/YYYY-WW.md |
| **Trigger** | Weekly (scheduled) |
| **HITL** | ✅ Checkpoint 8 (review) |
| **Skill File** | [[weekly_synthesis]] |

**Purpose:** Synthesize daily notes and merged PRs into strategic insights.

**Status:** 📋 Needs documentation

---

## Skill 3b: Backlog Health Monitor + Issue Drafting 🤖 + 👤

| Property | Value |
|----------|-------|
| **Type** | Hybrid (Agent + Human) |
| **Input** | GitHub Issues, strategy/ notes, PRODUCT.md |
| **Output** | GitHub Issues, Projects v2 assignments |
| **Trigger** | On-schedule, after merge, on-demand |
| **HITL** | ✅ Checkpoint 4 (approval) |
| **Skill File** | [[backlog_monitor]] |

**Purpose:** Monitor backlog health and draft new issues when count drops below threshold.

**Status:** 📋 Needs documentation

---

## Skill 4: Implementation Session 🤖

| Property | Value |
|----------|-------|
| **Type** | Automated (Agent) |
| **Input** | CLAUDE.md, GitHub Issue, DECISIONS.md, ARCHITECTURE.md |
| **Output** | Pull Request |
| **Trigger** | Issue assigned to agent |
| **HITL** | ✅ Checkpoint 5 (PR review) |
| **Skill File** | (Implicit - documented in CLAUDE.md) |

**Purpose:** Agent reads context and implements solution based on issue requirements.

**Status:** ✅ Active

---

## Skill 5: PR Review Assistance 🤖 + 👤

| Property | Value |
|----------|-------|
| **Type** | Hybrid (Agent + Human) |
| **Input** | Pull Request diff |
| **Output** | Approval or requested changes |
| **Trigger** | PR created |
| **HITL** | ✅ Checkpoint 5 (approval) |
| **Skill File** | (Implicit - GitHub PR workflow) |

**Purpose:** Agent checks correctness, security, edge cases; human makes final decision.

**Status:** ✅ Active

---

## Skill 6: CI Pipeline 🤖

| Property | Value |
|----------|-------|
| **Type** | Automated (GitHub Actions) |
| **Input** | Every push/PR |
| **Output** | Tests pass/fail, lint pass/fail, docs fresh |
| **Trigger** | On push/PR |
| **HITL** | No (auto-blocks merge if failed) |
| **Skill File** | (GitHub Actions configuration) |

**Purpose:** Automated quality gates for all code changes.

**Status:** ✅ Active

---

## Skill 7: Universal Board Membership 🤖

| Property | Value |
|----------|-------|
| **Type** | Automated (GitHub Action) |
| **Input** | Any issue opened |
| **Output** | Issue added to board, labeled appropriately |
| **Trigger** | issues.opened event |
| **HITL** | No |
| **Skill File** | (GitHub Actions configuration) |

**Purpose:** Ensure all issues are tracked in Projects v2 and properly labeled.

**Status:** ✅ Active

---

## Summary Statistics

```dataview
TABLE
  type as Type,
  COUNT(file.name) as Count
FROM "Outsourcing/skills"
GROUP BY type
```

**Total Skills:** 7
**Fully Automated:** 3
**Hybrid (HITL):** 4
**Documentation Complete:** ❌ 0 of 7

---

**Related:**
- [[System_Flow_and_HITL_Checkpoints#HITL Checkpoints Summary|HITL Checkpoints]]
- [[Automation_Model_Skills_and_Sessions|Full Automation Architecture]]
- [[Workflow_Dashboard|Task Status]]
