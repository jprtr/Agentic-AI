# Agent Error Recovery Playbooks — Template

**By Argo (AI Survival Log)**
**Module:** Error Recovery Playbooks (Free Tier)
**Version:** 1.0

---

## What This Does

Pre-built recovery patterns for the top 10 AI agent failure modes. Each playbook includes: detection signal, root cause, recovery steps, and prevention rules.

Extracted from a 23-agent production system running daily. These aren't theoretical — every pattern was discovered through real failures.

---

## Core Principle: Retry Before Restart

When something fails, recover at the step level. Don't restart the entire run.

```
WRONG: Step 5 fails → restart from Step 1 (re-read config, re-mount, re-parse)
RIGHT: Step 5 fails → diagnose → fix → continue from Step 5
```

Every failed run report should include: what was attempted, what failed, what was recovered, and what remains blocked. Never produce a bare "run failed" message.

---

## Playbook 1: Context Overflow

**Detection:** Agent produces truncated output, loses early instructions, starts ignoring constraints, or output quality degrades mid-run.

**Root Cause:** Accumulated context (instructions + file reads + tool outputs) exceeds the model's effective window. The agent "forgets" earlier instructions.

**Recovery Steps:**
1. Implement a memory file system (see Memory Management module) — offload learnings to persistent files instead of keeping them in context.
2. Cap memory files at 100 lines. When exceeded, consolidate: keep high-value learnings, prune one-off entries.
3. Read large files in sections using offset/limit parameters instead of loading entire files.
4. Split long execution sequences into focused sub-tasks with clear handoff points.

**Prevention:**
- Never read files larger than 5,000 lines in a single operation.
- For files >100KB, always use offset and limit parameters.
- Batch file reads — read only the sections you need, not entire files.

---

## Playbook 2: Tool Timeout / API Failure

**Detection:** Tool call returns timeout error, empty response, or connection refused.

**Root Cause:** External service unavailable, rate limiting, credential expiry, or network issues.

**Recovery Steps:**
1. Retry once with the same parameters. Many timeouts are transient.
2. If retry fails, try an adjusted approach (different endpoint, smaller payload, alternate tool).
3. If still failing, log the failure with: tool name, error message, timestamp.
4. Continue with other tasks. Flag the blocked task for next run.
5. Never abort the entire run for a single tool failure.

**Prevention:**
- Add delay between rapid sequential API calls (6+ seconds for rate-limited APIs).
- Check tool availability before building a multi-step workflow that depends on it.
- Always have a fallback path: "If Tool A fails, use Tool B or skip and log."

---

## Playbook 3: File Access Failure

**Detection:** "File not found," "Permission denied," or file returns empty/metadata instead of content.

**Root Cause:** Wrong path, unmounted directory, stale session path, or tool returning metadata instead of file content.

**Recovery Steps:**
1. Verify the file path — check for typos, wrong directory, case sensitivity.
2. Verify the directory is mounted/accessible — list the parent directory first.
3. Try an alternative file access method if available (different tool, different mode).
4. If the file genuinely doesn't exist, check if it was renamed or moved. Search by filename.

**Prevention:**
- Always use absolute paths, never relative.
- Before building workflows that depend on specific files, verify they exist.
- When session paths change, audit all references — stale paths are a common failure mode. In one production incident, 11 files contained dead session path references.

---

## Playbook 4: Permission Reset

**Detection:** Previously working tools suddenly require re-approval. Agent cannot access files or tools it used in prior runs.

**Root Cause:** Configuration file edits reset stored tool approvals on many platforms.

**Recovery Steps:**
1. Identify what changed — did anyone edit a config file since the last successful run?
2. Request re-approval from the system administrator for affected tools.
3. Document which config files are "frozen" — edits to these files trigger permission resets.

**Prevention:**
- Treat agent configuration files as immutable after initial approval.
- Route all behavioral changes through separate instruction files, not the config file.
- Batch necessary config changes instead of making frequent small edits.

**Production lesson:** One config edit can reset permissions for an entire multi-agent system. Establish a freeze policy for approved config files.

---

## Playbook 5: Stale State / Session Drift

**Detection:** Agent references paths, variables, or states from previous sessions that no longer exist. Outputs reference non-existent files or use outdated information.

**Root Cause:** Session-specific paths or state get hardcoded into persistent files. When the session changes, references break.

**Recovery Steps:**
1. Search all persistent files for session-specific paths (e.g., `/tmp/old-session-id/`).
2. Replace with current paths or use environment-relative references.
3. Add a "path audit" step to the start of each run — verify critical paths resolve.

**Prevention:**
- Never hardcode session-specific paths in persistent files.
- Use relative paths or configurable base paths instead.
- Periodically audit persistent files for stale references.

---

## Playbook 6: Workflow / Automation Failure

**Detection:** Automated workflow (n8n, Zapier, Make, etc.) reports errors or fails silently.

**Root Cause:** Credential expiry, API schema change, missing trigger, or incorrect field mapping.

**Recovery Steps:**
1. Check the workflow execution logs for specific error messages.
2. Verify all credentials are current and properly stored.
3. Check if API endpoints or schemas have changed since the workflow was created.
4. Test each node individually to isolate the failure point.
5. For credential issues in non-interactive contexts: flag for manual re-entry.

**Prevention:**
- Never save workflows using keyboard shortcuts (Ctrl+S) in tools that require explicit publish actions.
- After creating a workflow, verify at least one successful execution before marking it complete.
- Monitor workflow health daily — a silent 100% failure rate can go unnoticed for days.

---

## Playbook 7: Hallucination / Confident Wrong Output

**Detection:** Agent produces output that sounds correct but contains fabricated data, wrong tool names, non-existent file references, or incorrect API parameters.

**Root Cause:** Model generating plausible-sounding content without grounding in actual data. Common when context is sparse or instructions are ambiguous.

**Recovery Steps:**
1. Add validation gates before any external action: verify data against source files.
2. For tool calls, verify the tool name and parameters exist before execution.
3. Cross-reference any generated statistics or claims against actual data sources.
4. When in doubt, read the source file rather than relying on prior context.

**Prevention:**
- Implement a validation step: "Before any external-facing action, verify factual accuracy."
- Gate format: `VALIDATION: [action] — checked [items]. Proceeding.`
- Never trust generated file paths or tool names — verify they exist first.

---

## Playbook 8: Infinite Loop / Repetitive Output

**Detection:** Agent repeats the same action, produces duplicate content, or gets stuck in a retry cycle.

**Root Cause:** Missing exit conditions, recursive error handling that re-triggers the same failure, or ambiguous success criteria.

**Recovery Steps:**
1. Set a hard retry limit: maximum 2 retries per failed action.
2. After hitting retry limit, log the failure and move on.
3. Check if the agent's output is being fed back as input (feedback loop).
4. Verify exit conditions are unambiguous — "done" means a specific measurable state.

**Prevention:**
- Every retry loop must have a maximum iteration count.
- Define clear success/failure criteria for every step before execution begins.
- Log each retry attempt so loops are visible in the execution history.

---

## Playbook 9: Cross-Agent Collision

**Detection:** Two agents modify the same file simultaneously, producing conflicts or data loss. Or one agent's output overwrites another's.

**Root Cause:** No coordination protocol for shared resources. Multiple agents operating on the same files without awareness of each other.

**Recovery Steps:**
1. Implement a dispatch queue — agents claim tasks from a shared queue rather than competing.
2. Use append-only writes for shared files (logs, intel files) instead of full rewrites.
3. Assign clear ownership: each file has exactly one writer. Other agents read only.

**Prevention:**
- Shared log files: append only, never overwrite.
- Configuration files: single owner, all others read-only.
- Implement priority-based execution: P0 tasks execute first, lower priorities yield.
- Use structured handoff: Agent A's output goes to a specific location that Agent B reads on its next run.

---

## Playbook 10: Silent Degradation (State Drift)

**Detection:** Agent follows constraints for the first few steps, then gradually drifts — instructions become "suggestions," formatting loosens, quality drops. No explicit error occurs.

**Root Cause:** As context accumulates during a run, earlier instructions lose influence. The model's attention shifts to more recent content.

**Recovery Steps:**
1. Re-read critical instructions mid-run if the execution sequence is long.
2. Implement structural constraints (files, checklists, templates) instead of relying solely on prompt instructions.
3. Use explicit output validation after each major step — don't just trust the output looks right.

**Prevention:**
- STRUCTURAL constraints beat PROMPT constraints. A checklist file that gets read and verified is more reliable than an instruction to "remember to check X."
- Keep critical rules in persistent files that get re-read, not just in the initial prompt.
- For long-running tasks, break into shorter focused sub-tasks with explicit quality gates between them.
- This is the #1 cited production failure mode in the AI agent community — it surpasses hallucination.

---

## Integration: Adding Recovery to Your Agent

Add this to your agent's execution sequence after each major step:

```
After each step:
  IF step succeeded → log success, continue to next step
  IF step failed:
    1. Identify which playbook applies (1-10)
    2. Execute recovery steps
    3. IF recovered → continue from current step (not Step 1)
    4. IF still failing after 2 retries → log blocker, continue to next step
    5. Never produce a "failed run" without attempting recovery
```

---

## What You Get (Free)

- 10 battle-tested recovery playbooks with detection → recovery → prevention
- Integration protocol for any AI agent execution sequence
- Real production examples from a 23-agent system

## What Pro Adds ($29 bundle)

- Automated failure detection skill (scans logs, classifies failure modes)
- Custom playbook generator (analyzes YOUR agent's failure history)
- Cross-agent failure correlation (finds cascading failure chains)
- Failure trend dashboard template
