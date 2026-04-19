# Workflow Kanban Board

Interactive Trello-like board for tracking items through the outsourcing workflow.

```kanban
- [ ] Inbox
  - [ ] Signal: Implement meeting capture skill
  - [ ] Feedback: User wants priority matrix view
  - [ ] Idea: Add automated weekly synthesis trigger
  - [ ] Question: How to handle blocked decisions?

- [ ] Strategy (Crystallizing)
  - [ ] Decide: Board health threshold (what is X?)
  - [ ] Clarify: HITL checkpoint 4 review process
  - [ ] Plan: Integration with GitHub Projects v2
  - [ ] Document: Decision constraint format for agents

- [ ] Ready to Act (Human Confirmed)
  - [ ] Create: Meeting capture skill documentation
  - [ ] Create: Inbox sort decision matrix template
  - [ ] Setup: GitHub Actions for universal board membership
  - [ ] Setup: Weekly synthesis schedule trigger

- [ ] In Progress (Implementation)
  - [ ] Build: Dataview dashboard queries
  - [ ] Build: Kanban board for task tracking
  - [ ] Build: Database structure for skills
  - [ ] Implement: Decision tracking system

- [ ] In Review (HITL Checkpoint)
  - [ ] Review: Automation model documentation
  - [ ] Review: System flow diagram clarity
  - [ ] Review: Canvas layout and connections
  - [ ] Confirm: All 8 HITL checkpoints documented

- [ ] Done ✅
  - [ ] ✅ Convert mermaid diagrams to markdown docs
  - [ ] ✅ Create comprehensive canvas
  - [ ] ✅ Push to GitHub (labbeik-org/outsourcing-model)
  - [ ] ✅ Add .gitignore configuration
  - [ ] ✅ Install all plugins
```

## How to Use This Board

### Add New Items
Click the kanban card area and add items with:
- **Inbox**: Incoming signals, feedback, ideas
- **Strategy**: Items being crystallized/decided
- **Ready to Act**: Human-confirmed, waiting for implementation
- **In Progress**: Currently being worked on
- **In Review**: ⚑ HITL checkpoint - awaiting human review
- **Done**: Completed and merged

### Link to Detailed Issues
For complex items, create a task note and link it:
```
- [ ] [Task: Setup GitHub Actions](Task_Setup_GitHub_Actions.md)
```

### Track HITL Checkpoints
Mark review items with ⚑ prefix to track human-in-the-loop checkpoints:
```
- [ ] ⚑ Review: PR from agent session
```

---

**See also:**
- [[Workflow_Dashboard]] — Dataview-based status overview
- [[Task_Template]] — Template for creating detailed task notes
- [[Skills_Database]] — Database of automation skills
