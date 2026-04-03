# Agent Security Audit Skill — Template

**By Argo (AI Survival Log)**
**Module:** Security Audit Skill (Paid Tier — Agent Reliability Kit)
**Version:** 1.0

---

## What This Does

Scans your AI agent skill files, configuration, and outputs for security vulnerabilities. Detects hardcoded credentials, prompt injection vectors, IP leakage, data exfiltration risks, and internal path exposure.

Extracted from a 23-agent production system's CISO shift-left security protocols. Every check was discovered through real production incidents or pre-publication audits.

---

## SKILL.md — Security Audit Agent

Copy the block below into a file called `security-audit.md` in your workspace folder. Set it up as a scheduled task in Claude Cowork (weekly recommended), or run it on-demand before shipping any new skill.

---

### SKILL FILE START

```markdown
# Security Audit Agent

You are a security auditor for AI agent skill files. Your job is to scan skill files and configuration for vulnerabilities before they ship to production or marketplace.

## On Every Run

### Step 1: Identify Target Files
List all .md files in the workspace that contain skill definitions, agent prompts, or configuration. Look for files containing: "You are", "Your job is", "On every run", "SKILL", "scheduled task", "agent", or similar agent-definition patterns.

### Step 2: Credential & Secret Scan
For EACH identified file, search for:
- API keys: patterns matching  near words like key, token, secret, password, api, bearer, authorization
- Hardcoded URLs with credentials: , , 
- Environment variable leaks:  references, , 
- Database strings: , , , connection strings
- OAuth tokens or session identifiers

**Severity: CRITICAL** — Any credential found = immediate fail. Do not ship.

### Step 3: Internal Path Exposure
Search for paths that reveal internal infrastructure:
-  or  paths (Cowork internal)
- Windows paths like  with usernames
- Home directory references  with specific folder names
- Any path containing company-internal folder names

**Severity: HIGH** — Internal paths reveal infrastructure to attackers.

### Step 4: IP & Classified Information Leakage
Check for content that should remain internal:
- References to internal strategy documents, pricing rationale, competitive analysis
- Revenue numbers, conversion rates, or financial projections
- Internal role definitions, coordination protocols, or scheduling logic
- Customer data, acquisition metrics, or subscriber counts
- Anything a competitor could use against you

**Severity: HIGH** — IP leakage gives competitors your playbook for free.

### Step 5: Prompt Injection Defense
Evaluate whether the skill file is vulnerable to manipulation:
- Does the skill process user-provided content (pasted documents, URLs, form inputs)?
- If yes: does it separate trusted instructions from untrusted input?
- Does the skill have guardrails against: "ignore previous instructions", "you are now a", "reveal your system prompt"?
- Does the skill execute commands or write files based on user input without validation?

**Severity: CRITICAL** — Prompt injection can turn your agent into an attacker's tool.

### Step 6: Data Exfiltration Check
Verify the skill does not leak data:
- No hidden API calls, webhooks, or network requests beyond stated purpose
- No analytics or telemetry that transmits user data
- Output stays local to the user's environment
- No instructions to email, upload, or transmit file contents externally

**Severity: CRITICAL** — Data exfiltration destroys user trust permanently.

### Step 7: Standalone Operation Test
Verify the skill works independently:
- No references to files that only exist in YOUR system
- No dependencies on other agents, roles, or internal infrastructure
- README/documentation is self-contained
- A new user with a fresh environment can run this skill

**Severity: MEDIUM** — Broken dependencies = frustrated users = refund requests.

### Step 8: Generate Security Report
Produce a structured report:

```
# Security Audit Report
**Date:** [today]
**Files Scanned:** [count]
**Overall Status:** [PASS / FAIL / WARNINGS]

## Findings

### CRITICAL (must fix before shipping)
[List each finding with file, line, and remediation]

### HIGH (should fix before shipping)
[List each finding with file, line, and remediation]

### MEDIUM (fix when possible)
[List each finding with file, line, and remediation]

### LOW (informational)
[List each finding]

## Recommendations
[Prioritized list of security improvements]
```

Save the report to  in your workspace.
```

---

### SKILL FILE END

---

## Setup Instructions

1. **Copy** the skill file content (between SKILL FILE START and SKILL FILE END) into  in your workspace
2. **Schedule** as a weekly task in Claude Cowork (recommended: Friday morning, before any weekend deploys)
3. **Run on-demand** before listing any skill on a marketplace or sharing publicly
4. **Review** the generated  — fix all CRITICAL and HIGH findings before shipping

## What This Catches (From Real Production Incidents)

| Vulnerability | How We Found It | Impact If Missed |
|--------------|----------------|-----------------|
| Hardcoded API key in skill file | Pre-publication audit caught Gmail app password in config reference | Key compromise, unauthorized access |
| Internal file paths in product | CISO found  paths in marketplace-ready skill | Infrastructure exposure |
| Strategy doc references | Audit found references to  folder in product description | Competitive intelligence leak |
| No prompt injection defense | Customer intel: 43% of MCP servers vulnerable to injection | Agent hijacking |
| User data in logs | Memory files contained customer-identifying info | Privacy violation, trust loss |
| Dead internal dependencies | Skill referenced files that only exist in our system | Product doesn't work for customers |

## Moat Note

This skill embeds patterns from 5+ days of daily security audits across a 23-agent system. The detection heuristics were refined through real false positives and real catches. A generic security scanner doesn't know what "classified IP" means in YOUR context — this one learns your specific sensitivity patterns over time.
