# Hybrid team setup

Operating model for a solo human (PM, PO, CTO, SWE, Delivery Manager, Code Reviewer) working alongside Claude Code agents across several projects.

![Architecture](./hybrid_team_setup_final.svg)

## Core principle

Everything is a Git repo with a `CLAUDE.md`. Context lives next to the work it describes. Agents are a capability woven through every tier, not a terminal endpoint. The system compounds because shipped work feeds back into strategic thinking via a weekly synthesis loop.

## The tiers

### External world
Users, meetings, market signals, customer feedback. The raw material the system processes. You are the filter that turns external reality into vault-worthy signal.

### You
The single point of coordination. Wears six hats. Routes between context surfaces, writes briefs, reviews PRs, drives synthesis. The bottleneck worth protecting.

### Context layer — Git repositories

Two structurally identical families, different scopes.

**`vault` (private repo)** — personal brain, cross-project.
- `CLAUDE.md` — how the vault is organized, conventions, reading order for agents
- `daily/` — daily notes, chronological
- `strategy/` — synthesized thinking per project/area
- `decisions/` — your decisions and reversals
- `skills/` — custom Claude Code skills (weekly review, meeting capture, research)
- `inbox/` — pressure valve, sorted weekly

**project repos** — one per product, execution context.
- `CLAUDE.md` — how to work in this codebase, conventions, test expectations
- `ARCHITECTURE.md` — why the system is shaped this way
- `DECISIONS.md` — ADR-style log of non-obvious choices
- `PRODUCT.md` — current priorities, roadmap, the why
- `skills/` — project-specific Claude Code skills
- `src/` — the code itself

Translation between vault and project repos is a deliberate human-driven act (or a skill invocation), not an automatic sync. Strategic thinking from the vault graduates into a project's `DECISIONS.md` when it crystallizes.

### GitHub — remote + execution primitives
Hosts every repo above (including the vault). Provides:
- **Issues** — self-contained briefs, written from vault context, readable by agents
- **Projects v2** — roadmap view across repos (the lightweight alternative to Linear)
- **Pull requests** — agent opens, you review
- **Actions** — CI, checks, doc-freshness gates
- **Merge** — the moment to update `DECISIONS.md`, `ARCHITECTURE.md`, `CHANGELOG.md`, `README.md` while context is warm

### Weekly synthesis
A Claude Code skill that reads the week's daily notes, meeting records, and merged PRs, then produces a synthesis note back into `strategy/`. This closes the loop from execution to strategy. Without it, the vault decays into a record of what you were thinking, not what you are thinking.

## Agents as a side rail

Claude Code agents operate at every tier, not just at execution. Two modes:

- **Primary work (solid arrows)** — agents do the thing: implementing issues into PRs, running the weekly synthesis
- **Assistance (dashed arrows)** — agents help you: extracting notes from voice memos, distilling strategy into briefs, reviewing PRs, proposing doc updates on merge

The discipline: agents read your shared brain (both vault and project repos), but they don't decide what matters. You do the filtering, the deciding, the reviewing. They do the capture, the execution, the synthesis drafting.

## Why not Linear

Linear would split the invariant that everything-is-a-repo-with-a-CLAUDE.md. It would:
- Add a non-Git, non-markdown tier that agents can't natively read
- Create two sources of truth (Linear + GitHub) requiring sync
- Split your brain between "strategic thinking in vault" and "tactical tracking in Linear" with fuzzy boundaries
- Add translation hops between format surfaces

Re-evaluate if: a human teammate joins who won't touch Git, external stakeholders need polished roadmap sharing monthly+, or a cycle cadence emerges with genuine participants.

## Flow of one piece of work

1. Signal captured from external world → vault `inbox/` or `daily/`
2. Matures into thinking in vault `strategy/` or `projects/`
3. Distilled into a self-contained issue brief
4. Issue lives on GitHub, tracked in Projects v2
5. Agent session picks it up, reads repo `CLAUDE.md`, implements, opens PR
6. You review (agents assist with checks)
7. Merge → updates `DECISIONS.md` and `ARCHITECTURE.md` while context is warm
8. Weekly synthesis skill reads the week's merges and updates vault `strategy/` — loop closed

## Where leverage compounds

Rank-ordered by how much each intervention pays back:

1. **Vault `CLAUDE.md` + `skills/`** — read every vault session, enables automation, compounds across projects
2. **Weekly synthesis skill** — closes the loop from execution back to strategy; without it, the system flatlines
3. **Issue brief quality** — determines agent output quality; the single highest-leverage PM act
4. **Repo `DECISIONS.md` discipline at merge time** — prevents agents from undoing deliberate choices three months later
5. **External signal filter** — "will I or an agent benefit from this in six months" keeps the vault dense

Everything else is plumbing.

## Cost stack

- Obsidian: free
- Claude Pro: ~€20/month
- GitHub: free for private repos, paid for Actions minutes at scale
- MCP servers (Obsidian, memory): free, self-hosted
- Total: ~€20/month for the whole setup
