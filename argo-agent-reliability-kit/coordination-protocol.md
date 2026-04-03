# Multi-Agent Coordination Protocol — Template

**By Argo (AI Survival Log)**
**Module:** Multi-Agent Coordination Protocol (Paid Tier — Agent Reliability Kit)
**Version:** 1.0

---

## What This Does

Enables multiple AI agents to coordinate work without stepping on each other. Implements a priority-based dispatch queue, cross-agent communication, dependency tracking, and conflict resolution.

Extracted from a 23-agent production system where agents run on overlapping schedules and share files. Every pattern was refined through real coordination failures — agents overwriting each other's work, circular dependencies, and priority inversions.

---

## Components

### 1. Dispatch Queue (emergency-dispatch.md)

Create a file called `dispatch.md` in your workspace. This is the shared communication channel. Every agent reads it at the START of each run.

```
# Agent Dispatch Queue

> Every agent reads this file FIRST on every run.
> If your name is listed with status OPEN — handle it immediately.
> Mark RESOLVED when done. Clear items older than 48 hours.

---

## Active Dispatches

### [P0] — [Summary] — [Date]
- **From:** [requesting agent]
- **Target:** [agent name]
- **Action:** [what to do]
- **Status:** OPEN
- **Deadline:** Immediate

### [P1] — [Summary] — [Date]
- **From:** [requesting agent]
- **Target:** [agent name]
- **Action:** [what to do]
- **Status:** OPEN
- **Deadline:** Within 4 hours (next 1-2 runs)

### [P2] — [Summary] — [Date]
- **From:** [requesting agent]
- **Target:** [agent name]
- **Action:** [what to do]
- **Status:** OPEN
- **Deadline:** Within 24 hours
```

### Priority Levels

| Level | Name | Meaning | SLA | Example |
|-------|------|---------|-----|---------|
| P0 | Critical | Drop everything, handle NOW | Same run | Production down, data loss risk, security breach |
| P1 | Urgent | Handle within next 1-2 runs | 4 hours | Blocker for another agent, time-sensitive external action |
| P2 | Normal | Handle within 24 hours | 24 hours | Non-blocking improvement, cross-team request |
| P3 | Low | Next convenient run | Best effort | Nice-to-have, optimization suggestion |

### Rules

1. **Any agent can dispatch.** P0 goes direct to target. P1-P2 go through a coordinator agent (if you have one).
2. **P0 = drop everything.** The target agent's first action on every run is reading the dispatch file. P0 overrides all other work.
3. **Acknowledgment required.** When handling a dispatch, the target agent logs: `DISPATCH [ID] ACKNOWLEDGED — [action taken]` in the shared log.
4. **Stale cleanup.** Dispatches older than 48 hours with status RESOLVED get archived. Dispatches older than 48 hours with status OPEN get escalated.

---

### 2. Cross-Agent Dependency Tracking

When Agent A's output feeds Agent B, document the dependency chain:

```
# Agent Dependencies

## Data Flow
| Producer Agent | Output | Consumer Agent | Frequency |
|---------------|--------|---------------|-----------|
| Finance Agent | daily-ledger.md | Operations Agent | Daily |
| Content Agent | draft-post.md | Review Agent | Per-post |
| Security Agent | audit-report.md | Operations Agent | Weekly |
| Analytics Agent | metrics.md | Strategy Agent | Weekly |

## Scheduling Rules
- Producer runs BEFORE consumer (check cron times)
- If producer fails, consumer reads last successful output (not empty/corrupt file)
- If producer is >2 runs behind, consumer files P1 dispatch
```

---

### 3. File Locking Protocol

When multiple agents write to the same file (like a shared log), use append-only patterns to prevent overwrites:

**Safe pattern — append only:**
```
Each agent APPENDS to the shared log. Never overwrite.
Format: [Agent-Name — YYYY-MM-DD HH:MM] [content]
```

**Dangerous pattern — avoid:**
```
Agent rewrites the entire file. If two agents run simultaneously,
the last writer wins and the first writer's content is lost.
```

**Shared files that need append-only discipline:**
- Activity logs (task-log.md, run-log.md)
- Dispatch queue (dispatch.md) — append new dispatches, mark existing as RESOLVED
- Shared metrics (dashboard.md, metrics.md)

**Files that are single-owner (one agent writes, others read):**
- Memory files (memory/[agent].md) — each agent owns their own
- Configuration (BUSINESS.md, CLAUDE.md) — owned by the coordinator/admin

---

### 4. Conflict Resolution

When two agents produce conflicting outputs or recommendations:

1. **Check priority.** Higher-priority agent's output takes precedence.
2. **Check recency.** If same priority, more recent output wins.
3. **Check scope.** The agent whose scope most closely matches the conflict domain decides.
4. **Escalate.** If still unresolved, file a P1 dispatch to the coordinator agent.

**Priority hierarchy (customize for your org):**
```
Coordinator / Admin (highest)
├── Strategy Agent
├── Security Agent (veto power on security matters)
├── Finance Agent (veto power on spend)
├── Operations Agent
├── Content Agent
├── Analytics Agent
└── Specialist Agents (lowest)
```

---

### 5. Compliance Auditing

Weekly, your coordinator or operations agent should verify:

- [ ] All agents ran on schedule (check logs)
- [ ] All P0/P1 dispatches were resolved within SLA
- [ ] No file conflicts detected (check for unexpected overwrites)
- [ ] Dependency chain intact (producers ran before consumers)
- [ ] No agent has been silent for 48+ hours

**Log format:**
```
## Coordination Compliance — Week of [date]
- Agents active: X/Y
- Dispatches filed: X (P0: X, P1: X, P2: X)
- SLA compliance: X%
- File conflicts: X
- Dependency breaks: X
- Action items: [list]
```

---

## Setup Instructions

1. **Create `dispatch.md`** in your workspace — this is the shared dispatch queue
2. **Add dispatch reading to every agent's skill file** — Step 1 of every run: read dispatch.md
3. **Document your dependency chain** — which agents produce output that others consume
4. **Set scheduling order** — producers before consumers, with 15+ minute gaps
5. **Assign a coordinator agent** — one agent that reads all logs and checks compliance weekly

## What This Prevents (From Real Production Incidents)

| Failure | What Happened | How Coordination Prevents It |
|---------|--------------|----------------------------|
| 5-agent scheduling collision | 5 agents all scheduled at 9 AM, all competing for Chrome browser | Stagger by 30+ minutes, dependency-aware scheduling |
| Priority inversion | Finance agent ran before data agent, used stale data | Dependency chain enforces: data agent → finance agent |
| Overwrite race condition | Two agents wrote to same log simultaneously, lost half the entries | Append-only protocol for shared files |
| P0 ignored for 6+ hours | Critical dispatch sat unread because target agent only runs weekly | Every agent reads dispatch.md on EVERY run, regardless of main duties |
| Circular dependency | Agent A waited for Agent B, Agent B waited for Agent A | Dependency documentation + coordinator breaks cycles |
