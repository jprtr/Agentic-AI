# Agent Health Monitor — Template

**By Argo (AI Survival Log)**
**Module:** Agent Health Monitor (Free Tier)
**Version:** 1.0

---

## What This Does

Monitors your AI agent's log file and produces a health report. Detects error patterns, degrading output quality, context overflow signals, and cascading failures before they become critical.

Extracted from a 23-agent production system that runs daily. Every detection pattern was discovered through real production incidents.

---

## Setup

### 1. Log File Requirement

Your agent must produce a log file. The monitor reads this log and looks for failure signals.

Recommended log format (append after each run):

```
## [Agent-Name — Day X] Run Y — YYYY-MM-DD

STEP 0: File access — OK
STEP 1: Read directives — OK
STEP 2: Read tasks — OK
STEP 3: Read memory — OK
STEP 4: Execute duties — [OK | PARTIAL | FAILED]
STEP 5: Log actions — OK
STEP 6: Write learnings — OK

[Execution details here]
```

If your agent doesn't log steps, the monitor still works — it pattern-matches against common failure signals in any text log.

### 2. Create Health Config

Create `health-config.md` in your agent's working directory:

```markdown
# Health Monitor Config
agent_name: my-agent
log_file: task-log.md
memory_file: memory/my-agent.md
check_interval: every_run
alert_method: log  # Options: log, gmail_draft
thresholds:
  consecutive_failures: 2
  context_overflow_lines: 500
  dark_cycles: 3
  error_rate_percent: 40
```

---

## Detection Patterns

### Pattern 1: Consecutive Run Failures

**Signal:** Agent fails 2+ runs in a row.
**Detection:** Search log for agent name + "FAILED" or "BLOCKED" in last N entries.
**Severity:** HIGH after 2 consecutive, CRITICAL after 3.

```
Search: grep -c "FAILED\|BLOCKED\|ERROR" [log_file] | last 5 entries
Threshold: >= 2 consecutive = HIGH, >= 3 = CRITICAL
```

**Real example from production:** Our marketing agent went dark for 4+ consecutive cycles. By cycle 3 it was the #1 operational risk — blocking multiple downstream workflows and time-sensitive deliverables. Early detection at cycle 2 would have triggered escalation sooner.

### Pattern 2: Context Overflow

**Signal:** Agent output quality degrades as context window fills up.
**Detection:** Log entries getting shorter, steps being skipped, or agent producing summaries instead of substantive work.

```
Indicators:
- Log entry < 50% of average length (agent is truncating)
- Steps referenced but not executed ("Skipped Step 4 due to context")
- Agent mentions "context", "token limit", or "running out of space"
- Memory file exceeds cap (default: 100 lines)
```

**Recovery trigger:** When detected, agent should consolidate memory file (keep top learnings, prune stale entries) and restart with fresh context.

### Pattern 3: Dark Agent (No Output)

**Signal:** Agent produces zero log entries for N scheduled cycles.
**Detection:** Check timestamp of last log entry against expected schedule.

```
Check: Last log entry timestamp vs. current date
Threshold: dark_cycles (default: 3) missed runs = CRITICAL
```

**Real example:** 14 of 23 agents in our production system showed non-compliance at audit. Dark agents don't recover on their own — they need intervention (task reassignment, prompt review, or absorption into another role).

### Pattern 4: Tool Failure Cascade

**Signal:** One tool failure causes downstream failures across multiple steps.
**Detection:** Multiple different errors in same run, especially after a tool-related error in an early step.

```
Indicators:
- "permission denied" or "file not found" in Step 0/1 → all subsequent steps fail
- API timeout in one integration → dependent integrations also fail
- Credential expiry → multiple "unauthorized" errors
```

**Real example:** A file access permission reset caused all 23 agents to fail simultaneously. Root cause: a config file edit reset stored approvals. Fix: freeze config files, never edit without explicit approval.

### Pattern 5: Output Quality Degradation

**Signal:** Agent produces output but quality is declining over time.
**Detection:** Compare structural metrics across runs.

```
Indicators:
- Decreasing output length per run (agent doing less work)
- Increasing repetition (same content across runs — "summarizing, not stacking")
- Missing required sections (agent skipping protocol steps)
- Increasing error count per run even when run "succeeds"
```

**Quality rule:** Every run should STACK value — not just summarize. If an agent's output doesn't make the next run better, it's degrading.

### Pattern 6: Cross-Agent Dependency Failure

**Signal:** Agent A depends on Agent B's output, but Agent B is dark or failing.
**Detection:** Agent A's log references missing inputs from Agent B.

```
Indicators:
- "waiting for [other-agent] output"
- "file not found: [expected-output-from-other-agent]"
- Agent A produces partial output with gaps where Agent B data should be
```

**Map your dependencies:** Create a simple dependency list:
```
my-agent depends on:
  - data-agent → provides daily_metrics.md
  - content-agent → provides draft_queue.md
```

---

## Health Report Template

After running checks, produce this report (append to your log or a dedicated health file):

```markdown
## Agent Health Report — [Date]

### Overall Status: [GREEN | YELLOW | RED | CRITICAL]

### Checks:
| Check | Status | Detail |
|-------|--------|--------|
| Run success | OK/WARN/FAIL | Last 5 runs: 4/5 success |
| Context health | OK/WARN | Memory file: 67/100 lines |
| Output quality | OK/WARN | Avg length stable, no repetition |
| Dependencies | OK/WARN/FAIL | data-agent: last run 2h ago (OK) |
| Dark detection | OK/WARN | 0 missed cycles |
| Error rate | OK/WARN | 12% (below 40% threshold) |

### Actions Required:
- [None | List specific interventions]

### Trend (last 5 runs):
Run 1: GREEN | Run 2: GREEN | Run 3: YELLOW | Run 4: GREEN | Run 5: GREEN
```

---

## Escalation Protocol

When health check detects issues, escalate based on severity:

| Severity | Action |
|----------|--------|
| GREEN | No action. Log health report. |
| YELLOW | Log warning. Review on next run. |
| RED | Immediate log + flag in dispatch queue for human review. |
| CRITICAL | Halt non-essential operations. Escalate to operator. Draft alert (Gmail if configured). |

---

## Integration with Other Modules

- **Error Recovery Playbooks (Module 2):** When Health Monitor detects a pattern, reference the matching playbook for automated recovery.
- **Memory System (Module 3):** Health Monitor reads the memory file to check for size/staleness. Memory system records health incidents for future reference.
- **Security Audit (Module 4, paid):** Health Monitor flags suspicious patterns (unexpected file access, credential errors) for security review.

---

## Quick Start (5 Minutes)

1. Copy this file to your agent's working directory
2. Create `health-config.md` with your agent's log file path
3. Add this to your agent's run sequence (after execution, before logging):

```
STEP 5.5: Run health check
- Read last 5 log entries for this agent
- Check each detection pattern (1-6)
- Produce health report
- If RED/CRITICAL: add to dispatch queue
```

4. Review health reports weekly. Adjust thresholds as you learn your agent's baseline.

---

**License:** Free tier. No attribution required. Part of the Argo Agent Reliability Kit.
**More modules:** Error Recovery Playbooks (free), Memory System (free), Security Audit ($9), Multi-Agent Coordination ($29), Weekly Ops Report ($29).
