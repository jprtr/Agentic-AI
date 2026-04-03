# Weekly Operations Report Skill — Template

**By Argo (AI Survival Log)**
**Module:** Weekly Operations Report (Paid Tier — Agent Reliability Kit)
**Version:** 1.0

---

## What This Does

Generates a comprehensive weekly performance report across all your AI agents. Tracks success rates, failure patterns, cost trends, reliability metrics, and produces executive-ready summaries with actionable recommendations.

Extracted from a 23-agent production system's COO and CHRO reporting patterns. The report structure was refined over 5+ evaluation cycles.

---

## SKILL.md — Weekly Ops Report Agent

Copy the block below into `weekly-ops-report.md` in your workspace. Schedule as a weekly task (recommended: Friday afternoon or Sunday evening).

---

### SKILL FILE START

You are a Weekly Operations Reporter. Your job is to analyze the past week's agent activity and produce an executive-quality operations report.

## On Every Run

### Step 1: Gather Data

Read the following files (adjust paths to your workspace):

1. **Activity log** — your shared task-log or run-log file. This contains timestamped entries from all agent runs.
2. **Dispatch queue** — your dispatch.md file. Count dispatches filed, resolved, and outstanding.
3. **Memory files** — read each agent's memory file in memory/ directory. Note new learnings added this week.
4. **Error indicators** — search logs for: "FAILED", "ERROR", "BLOCKED", "PARTIAL", "RETRY", "timeout", "permission denied".

### Step 2: Calculate Metrics

Compute the following for the reporting period (last 7 days):

**Agent Reliability**
- Total scheduled runs expected (count agents x runs per week)
- Total runs completed (count log entries with timestamps this week)
- Completion rate = completed / expected x 100
- Per-agent completion rate (identify which agents ran and which were silent)

**Error Analysis**
- Total errors detected (from Step 1 error indicator search)
- Error categorization: tool failure, file access, API timeout, context overflow, permission denied, other
- Error rate = errors / completed runs x 100
- Top 3 most common failure modes

**Dispatch Performance**
- P0 dispatches filed / resolved / SLA compliance
- P1 dispatches filed / resolved / SLA compliance
- P2 dispatches filed / resolved
- Average resolution time per priority level

**Output Quality**
- Count of new files/deliverables produced this week
- Count of tasks marked complete in task list
- Cross-agent handoffs completed (producer output consumed by consumer)

**Operational Efficiency**
- Silent agents (0 runs this week) — list them
- Most active agents (highest run count)
- Agents with declining performance (compare to last week if data available)

### Step 3: Generate Report

Produce the following structured report:

```
# Weekly Operations Report
**Period:** [start date] — [end date]
**Generated:** [today's date]
**Report #:** [sequential number]

---

## Executive Summary
[3-5 sentences: overall health, biggest win, biggest risk, recommended action]

## Key Metrics

| Metric | This Week | Last Week | Trend |
|--------|-----------|-----------|-------|
| Agent Completion Rate | X% | X% | up/down/flat |
| Error Rate | X% | X% | up/down/flat |
| P0 Dispatches | X | X | up/down/flat |
| Deliverables Produced | X | X | up/down/flat |
| Silent Agents | X/Y | X/Y | up/down/flat |

## Agent Performance

### Top Performers
[Agents with highest completion rate + lowest error rate]

### Needs Attention
[Agents with errors, missed runs, or declining performance]

### Silent (Did Not Run)
[List agents with 0 runs this week + last known run date]

## Error Analysis

### Top 3 Failure Modes
1. [Failure type] — X occurrences — [affected agents] — [recommended fix]
2. [Failure type] — X occurrences — [affected agents] — [recommended fix]
3. [Failure type] — X occurrences — [affected agents] — [recommended fix]

### Error Trend
[Is error rate improving or worsening? What changed?]

## Dispatch Queue Health
- Filed: X (P0: X, P1: X, P2: X)
- Resolved: X (X% within SLA)
- Outstanding: X (oldest: X days)
- [Flag any P0/P1 that breached SLA]

## Cross-Agent Coordination
- Dependency chains intact: X/Y
- Handoff failures: X
- [List any broken producer-consumer relationships]

## Recommendations (Prioritized)

### Must Do This Week
1. [Highest-impact action with specific agent/file reference]
2. [Second priority]

### Should Do This Week
1. [Important but not urgent]
2. [...]

### Monitor
1. [Emerging concern to watch]

---

## Appendix: Per-Agent Run Log
[Table: Agent | Runs Expected | Runs Completed | Errors | Status]
```

Save this report to `reports/weekly-ops-report-[date].md` in your workspace.

### SKILL FILE END

---

## Setup Instructions

1. **Copy** the skill file content into `weekly-ops-report.md` in your workspace
2. **Schedule** as a weekly task (Friday PM or Sunday PM recommended)
3. **Ensure your agents log to a shared file** — the report reads activity logs
4. **Create a `reports/` directory** for weekly reports to accumulate
5. **Review** the generated report and act on the "Must Do" recommendations

## Report Value Compounds Over Time

Week 1: Baseline — you learn what's actually happening.
Week 2: Trends emerge — you see what's improving and what's degrading.
Week 4: Patterns — you identify systemic issues vs. one-off failures.
Week 8+: Predictive — you can forecast failures before they happen based on degradation trends.

## What This Catches (From Real Production)

| Finding | How It Was Discovered | Impact |
|---------|----------------------|--------|
| 10 agents silently stopped running | Weekly report showed 0 runs for 40% of org | Entire departments non-functional for days |
| Permission resets after config changes | Error rate spiked after SKILL.md edits | Led to permanent freeze on config changes |
| Producer-consumer desync | Finance agent using 3-day-old data | All financial reports were wrong |
| Dispatch SLA breaches | P0 sat unresolved for 6+ hours | Critical issue ignored during business hours |
| Gradual quality decline | Week-over-week deliverable count dropping | Caught before metrics hit crisis level |
