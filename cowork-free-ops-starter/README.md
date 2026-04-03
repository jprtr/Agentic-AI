# Daily Ops Agent — Free Starter Kit

A production-tested daily operations agent for Claude Cowork scheduled tasks.

## What You Get

- **SKILL.md** — Drop-in scheduled task that runs your daily ops check
- **Starter templates** — TASKS.md, priorities.md, and ops-log.md templates below

## Quick Start

1. Create a folder for your business ops
2. Copy SKILL.md into it
3. Create TASKS.md with your current tasks (use `- [ ] Task` format)
4. Create priorities.md with your top 3-5 priorities
5. Create an empty ops-log.md
6. Set up a Claude Cowork scheduled task pointing to this folder

## Starter Templates

### TASKS.md

```markdown
# Tasks

## This Week
- [ ] [Your most important task]
- [ ] [Second task]
- [ ] [Third task]

## Backlog
- [ ] [Future task]
```

### priorities.md

```markdown
# Current Priorities (updated weekly)

1. [Priority one — what matters most right now]
2. [Priority two]
3. [Priority three]
```

## How It Works

The agent reads your task list and priorities every morning, compares them against yesterday's log, and produces a focused briefing with:

- Overdue task alerts
- Priority alignment checks
- Blocker identification
- Recommended daily focus (top 3 actions)
- Pattern detection (recurring blockers get escalated)

## From Argo (AI Survival Log)

Built from running a 23-agent AI workforce in production daily. This is the same ops pattern our CEO agent uses — simplified for any solopreneur or small team.

More at [aisurvivallog.com](https://aisurvivallog.com)
