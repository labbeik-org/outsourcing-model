# Workflow Dashboard

Real-time overview of all tasks and their status using Dataview queries.

## 📊 Status Overview

```dataview
TABLE
  status as Status,
  assignee as Assigned to,
  priority as Priority,
  duedate as Due
FROM "Outsourcing/tasks"
WHERE status
GROUP BY status
SORT status ASC
```

## 🎯 Current Tasks by Stage

### Inbox (New Signals)
```dataview
LIST
FROM "Outsourcing/tasks"
WHERE status = "inbox"
SORT created DESC
LIMIT 10
```

### 🤔 Strategy (Crystallizing)
```dataview
LIST
FROM "Outsourcing/tasks"
WHERE status = "strategy"
SORT priority DESC
```

### ✅ Ready to Act (Confirmed)
```dataview
LIST
FROM "Outsourcing/tasks"
WHERE status = "ready"
SORT priority DESC
```

### ⚙️ In Progress (Implementation)
```dataview
LIST
FROM "Outsourcing/tasks"
WHERE status = "in-progress"
SORT priority DESC
```

### 🔍 In Review (HITL Checkpoints)
```dataview
LIST
FROM "Outsourcing/tasks"
WHERE status = "in-review"
SORT priority DESC
```

Count: `=dv.pages('"Outsourcing/tasks"').where(t => t.status === "in-review").length`

### ✨ Done (Completed)
```dataview
LIST
FROM "Outsourcing/tasks"
WHERE status = "done"
SORT completed DESC
LIMIT 20
```

---

## 📈 Statistics

| Metric | Count |
|--------|-------|
| **Total Tasks** | `=dv.pages('"Outsourcing/tasks"').length` |
| **Inbox Items** | `=dv.pages('"Outsourcing/tasks"').where(t => t.status === "inbox").length` |
| **In Progress** | `=dv.pages('"Outsourcing/tasks"').where(t => t.status === "in-progress").length` |
| **Awaiting Review** | `=dv.pages('"Outsourcing/tasks"').where(t => t.status === "in-review").length` |
| **Completed** | `=dv.pages('"Outsourcing/tasks"').where(t => t.status === "done").length` |

---

## 🚨 Blocked Items

```dataview
LIST
FROM "Outsourcing/tasks"
WHERE blocked = true
SORT priority DESC
```

---

## ⚑ HITL Checkpoints Pending

```dataview
LIST
FROM "Outsourcing/tasks"
WHERE checkpoint AND status != "done"
SORT checkpoint ASC
```

---

## 📅 Due This Week

```dataview
LIST
FROM "Outsourcing/tasks"
WHERE duedate >= date(today) AND duedate <= date(today) + duration("7 days")
SORT duedate ASC
```

---

## 👤 By Assignee

```dataview
TABLE
  status as Status,
  priority as Priority,
  duedate as Due
FROM "Outsourcing/tasks"
GROUP BY assignee
```

---

**Related:**
- [[Workflow_Kanban_Board]] — Visual Kanban view
- [[Skills_Database]] — All automation skills
- [[Task_Template]] — Create new tasks
