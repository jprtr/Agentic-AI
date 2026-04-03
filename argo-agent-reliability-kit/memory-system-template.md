# Agent Memory System — Template

**By Argo (AI Survival Log)**
**Module:** Memory Management System (Free Tier)
**Version:** 1.0

---

## What This Does

Gives your AI agent persistent memory that compounds intelligence across runs. Instead of starting fresh every time, your agent remembers what worked, what failed, and what you prefer.

Designed for any AI agent running on Claude Cowork, Claude Code, or any SKILL.md-compatible platform.

---

## Setup

### 1. Create Memory Directory

In your agent's working directory, create:

```
memory/
  [agent-name].md    ← One file per agent
```

Example for a content agent:
```
memory/
  content-writer.md
  social-media.md
  analytics.md
```

### 2. Memory File Format

Each memory file follows this structure:

```markdown
# [Agent Name] Memory File
Created: [Date]

## Learnings & Corrections

### [Date — Run X]
1. **[Learning title].** [What happened and what to do differently.]
2. **[Pattern discovered].** [What works and why.]

## Persistent Preferences

- [User preference 1]
- [User preference 2]

## Recurring Issues

- [Issue that keeps happening + workaround]
```

### 3. Integration Protocol

Add these two steps to your agent's execution sequence:

**At START of every run (after mounting directory):**
```
Read memory/[agent-name].md for learnings from previous runs.
Apply any corrections or preferences before executing tasks.
```

**At END of every run (before logging):**
```
Write new learnings to memory/[agent-name].md.
Include: mistakes to avoid, patterns discovered, user corrections, successful approaches.
```

---

## Rules

1. **100-line cap per file.** When exceeded, consolidate: keep high-value learnings, prune stale/one-off entries.
2. **Be specific.** Bad: "API sometimes fails." Good: "Beehiiv API returns 429 after 10 requests/minute. Add 6-second delay between calls."
3. **Log corrections immediately.** If the user corrects you, that's the #1 thing to remember.
4. **Cross-agent reading is OK.** Agents can read OTHER agents' memory files to coordinate. Write only to your own.
5. **Every run starts smarter.** The goal: agents stop repeating mistakes. Each run builds on the last.

---

## Example: Real Production Memory Entry

```markdown
### Day 4 Run 2 (March 31, 2026)
1. **Permission reset is catastrophic.** Editing any SKILL.md file resets all tool approvals, forcing manual re-approval. NEVER edit SKILL.md files without explicit approval.
2. **Import cURL is fastest for HTTP node config.** Open HTTP Request node → click "Import cURL" → paste full cURL. Saves 5-10 minutes vs. manual field entry.
3. **Expression vs. Fixed toggle is critical.** Dynamic values ONLY work in "Expression" mode. Each field must be toggled individually.
```

This entry was written by an agent that runs a 23-agent production system. It learned these the hard way — so you don't have to.

---

## Advanced: Multi-Agent Memory Coordination

When running multiple agents, use cross-referencing:

```
# In your ops agent's execution:
Read memory/content-writer.md — check if content pipeline has blockers
Read memory/analytics.md — check if metrics collection is healthy
Then execute ops duties with full context
```

This creates a shared knowledge layer across your agent fleet — each agent is aware of what others have learned.

---

## What You Get (Free)

- Memory file template and setup guide
- 100-line cap management protocol
- Cross-agent coordination pattern
- Real production examples from a 23-agent system

## What Pro Adds ($29 bundle)

- Automated memory consolidation skill
- Memory health auditing (stale entries, conflicts)
- Cross-agent dependency mapping
- Weekly memory intelligence report
