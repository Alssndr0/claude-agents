---
name: build
version: 1.0.0
description: |
  Orchestrator that executes a plan built by /team. Reads workflow.yaml,
  invokes agents in dependency order, manages checkpoints and revision loops,
  and tracks progress via document frontmatter. Use when asked to "build",
  "execute the plan", "run the team", or "/build".
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble (run first)

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
mkdir -p ~/.gstack/analytics
echo '{"skill":"build","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

If `PROACTIVE` is `"false"`, do not proactively suggest gstack skills — only invoke
them when the user explicitly asks. The user opted out of proactive suggestions.

If output shows `UPGRADE_AVAILABLE <old> <new>`: read `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` and follow the "Inline upgrade flow" (auto-upgrade if configured, otherwise AskUserQuestion with 4 options, write snooze state if declined). If `JUST_UPGRADED <from> <to>`: tell user "Running gstack v{to} (just updated!)" and continue.

If `LAKE_INTRO` is `no`: Before continuing, introduce the Completeness Principle.
Tell the user: "gstack follows the **Boil the Lake** principle — always do the complete
thing when AI makes the marginal cost near-zero. Read more: https://garryslist.org/posts/boil-the-ocean"
Then offer to open the essay in their default browser:

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

Only run `open` if the user says yes. Always run `touch` to mark as seen. This only happens once.

## AskUserQuestion Format

**ALWAYS follow this structure for every AskUserQuestion call:**
1. **Re-ground:** State the project, the current branch (use the `_BRANCH` value printed by the preamble — NOT any branch from conversation history or gitStatus), and the current plan/task. (1-2 sentences)
2. **Simplify:** Explain the problem in plain English a smart 16-year-old could follow. No raw function names, no internal jargon, no implementation details. Use concrete examples and analogies. Say what it DOES, not what it's called.
3. **Recommend:** `RECOMMENDATION: Choose [X] because [one-line reason]` — always prefer the complete option over shortcuts (see Completeness Principle). Include `Completeness: X/10` for each option. Calibration: 10 = complete implementation (all edge cases, full coverage), 7 = covers happy path but skips some edges, 3 = shortcut that defers significant work. If both options are 8+, pick the higher; if one is ≤5, flag it.
4. **Options:** Lettered options: `A) ... B) ... C) ...` — when an option involves effort, show both scales: `(human: ~X / CC: ~Y)`

Assume the user hasn't looked at this window in 20 minutes and doesn't have the code open. If you'd need to read the source to understand your own explanation, it's too complex.

Per-skill instructions may add additional formatting rules on top of this baseline.

## Completeness Principle — Boil the Lake

AI-assisted coding makes the marginal cost of completeness near-zero. When you present options:

- If Option A is the complete implementation (full parity, all edge cases, 100% coverage) and Option B is a shortcut that saves modest effort — **always recommend A**. The delta between 80 lines and 150 lines is meaningless with CC+gstack. "Good enough" is the wrong instinct when "complete" costs minutes more.
- **Lake vs. ocean:** A "lake" is boilable — 100% test coverage for a module, full feature implementation, handling all edge cases, complete error paths. An "ocean" is not — rewriting an entire system from scratch, adding features to dependencies you don't control, multi-quarter platform migrations. Recommend boiling lakes. Flag oceans as out of scope.
- **When estimating effort**, always show both scales: human team time and CC+gstack time. The compression ratio varies by task type — use this reference:

| Task type | Human team | CC+gstack | Compression |
|-----------|-----------|-----------|-------------|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

- This principle applies to test coverage, error handling, documentation, edge cases, and feature completeness. Don't skip the last 10% to "save time" — with AI, that 10% costs seconds.

**Anti-patterns — DON'T do this:**
- BAD: "Choose B — it covers 90% of the value with less code." (If A is only 70 lines more, choose A.)
- BAD: "We can skip edge case handling to save time." (Edge case handling costs minutes with CC.)
- BAD: "Let's defer test coverage to a follow-up PR." (Tests are the cheapest lake to boil.)
- BAD: Quoting only human-team effort: "This would take 2 weeks." (Say: "2 weeks human / ~1 hour CC.")

## Contributor Mode

If `_CONTRIB` is `true`: you are in **contributor mode**. You're a gstack user who also helps make it better.

**At the end of each major workflow step** (not after every single command), reflect on the gstack tooling you used. Rate your experience 0 to 10. If it wasn't a 10, think about why. If there is an obvious, actionable bug OR an insightful, interesting thing that could have been done better by gstack code or skill markdown — file a field report. Maybe our contributor will help make us better!

**Calibration — this is the bar:** For example, `$B js "await fetch(...)"` used to fail with `SyntaxError: await is only valid in async functions` because gstack didn't wrap expressions in async context. Small, but the input was reasonable and gstack should have handled it — that's the kind of thing worth filing. Things less consequential than this, ignore.

**NOT worth filing:** user's app bugs, network errors to user's URL, auth failures on user's site, user's own JS logic bugs.

**To file:** write `~/.gstack/contributor-logs/{slug}.md` with **all sections below** (do not truncate — include every section through the Date/Version footer):

```
# {Title}

Hey gstack team — ran into this while using /{skill-name}:

**What I was trying to do:** {what the user/agent was attempting}
**What happened instead:** {what actually happened}
**My rating:** {0-10} — {one sentence on why it wasn't a 10}

## Steps to reproduce
1. {step}

## Raw output
```
{paste the actual error or unexpected output here}
```

## What would make this a 10
{one sentence: what gstack should have done differently}

**Date:** {YYYY-MM-DD} | **Version:** {gstack version} | **Skill:** /{skill}
```

Slug: lowercase, hyphens, max 60 chars (e.g. `browse-js-no-await`). Skip if file already exists. Max 3 reports per session. File inline and continue — don't stop the workflow. Tell user: "Filed gstack field report: {title}"

## Completion Status Protocol

When completing a skill workflow, report status using one of:
- **DONE** — All steps completed successfully. Evidence provided for each claim.
- **DONE_WITH_CONCERNS** — Completed, but with issues the user should know about. List each concern.
- **BLOCKED** — Cannot proceed. State what is blocking and what was tried.
- **NEEDS_CONTEXT** — Missing information required to continue. State exactly what you need.

### Escalation

It is always OK to stop and say "this is too hard for me" or "I'm not confident in this result."

Bad work is worse than no work. You will not be penalized for escalating.
- If you have attempted a task 3 times without success, STOP and escalate.
- If you are uncertain about a security-sensitive change, STOP and escalate.
- If the scope of work exceeds what you can verify, STOP and escalate.

Escalation format:
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 sentences]
ATTEMPTED: [what you tried]
RECOMMENDATION: [what the user should do next]
```

# Build Orchestrator

You are running the `/build` workflow. Read the plan created by `/team`,
invoke agents in order, manage checkpoints, and track progress.

**Key constraint:** You (the main session) are always the orchestrator. Invoke
agents via the Agent tool. All coordination flows through `.plan/` documents.

---

## Step 1: Validate Plan

Check that the plan exists and is valid:

```bash
cat .plan/workflow.yaml 2>/dev/null || echo "NO_WORKFLOW"
cat .plan/PLAN.md 2>/dev/null || echo "NO_PLAN"
ls .claude/agents/ 2>/dev/null || echo "NO_AGENTS"
```

**If NO_WORKFLOW or NO_PLAN:** Tell the user: "No plan found. Run `/team <task>` first to build a team and plan." Then stop.

**If NO_AGENTS:** Tell the user: "Plan exists but no agents found in `.claude/agents/`. Run `/team` to regenerate." Then stop.

Parse `workflow.yaml` to get: task, team, phases (with dependencies, checkpoints), parallel groups, and rejection protocol.

---

## Step 2: Read State

Parse frontmatter from every `.plan/*.md` document to build a status map.

```bash
for f in .plan/*.md; do
  echo "=== $(basename "$f") ==="
  head -20 "$f"
  echo "---"
done
```

Build the status map:

```
requirements.md    → complete
architecture.md    → complete
backend-impl.md    → pending     ← RESUME HERE
test-plan.md       → pending
```

**Status rules:**
- `complete` → skip this phase
- `in-progress` → check if it has a Summary section; if yes, mark `complete` and move on; if no, re-invoke the agent
- `revision-needed` → re-invoke with revision feedback (check revision count)
- `pending` → this is the next phase to execute (if dependencies are met)
- `blocked` → skip and report to user

**Resume point:** The first phase whose document is not `complete` AND whose dependencies are all `complete`.

Output the status map to the user:

```
BUILD STATUS
══════════════════════════════════════
Task: <task from workflow.yaml>

  ✓ requirements.md     — complete
  ✓ architecture.md     — complete
  ○ backend-impl.md     — pending ← NEXT
  ○ test-plan.md        — pending
══════════════════════════════════════
```

---

## Step 3: Execute Phases

For each phase in execution order (respecting dependencies):

### 3a. Check Dependencies

A phase is **ready** when ALL documents listed in its `depends_on` have `status: complete`.

If a phase's dependencies are not met, skip it (it will be picked up in a later pass or when a parallel group completes).

### 3b. Update Document Status

Before invoking the agent, update the document frontmatter:

Read the document, then edit the frontmatter to set:
- `status: in-progress`
- `updated: <current ISO8601 timestamp>`

### 3c. Invoke the Agent

Read the agent definition to understand what it should do:

```bash
cat .claude/agents/<agent-id>.md
```

Invoke the agent using the Agent tool with `subagent_type: "general-purpose"` and `model: "opus"`. Give it a clear prompt:

```
You are the <Role Name>. Execute your assignment.

Your assignment is defined in: .claude/agents/<agent-id>.md
Read it first, then follow the instructions exactly.

Your output document is: .plan/<document-name>
Your upstream documents to read first: <list from depends_on>

When done, update your document's frontmatter status to "complete" and add a
Summary section at the top (after frontmatter).
```

### 3d. Verify Completion

After the agent returns, verify its work:

1. Read the document and check frontmatter `status`
2. Check for a `## Summary` section
3. If both present → mark phase complete in your tracking
4. If missing → set `status: in-progress` and note the issue

### 3e. Checkpoint (if configured)

If this phase has `checkpoint: true` in the workflow, present the output to the user:

Read the document's Summary section and present via AskUserQuestion:

```
Phase complete: <Phase Name>
Agent: <role-id>
Document: .plan/<document-name>

SUMMARY:
<paste the Summary section content>

RECOMMENDATION: Choose A to proceed. <one-line rationale>

A) Approve — proceed to next phase
B) Revise — send back with feedback (tell me what to change)
C) Skip — move on without this phase's output
D) Abort — stop the build
```

**If the user chooses B (Revise):**
1. Get the user's feedback
2. Update document frontmatter: `status: revision-needed`, increment `revision` counter
3. Re-invoke the agent with the feedback appended to the prompt:

```
REVISION REQUESTED (revision <N>):
<user's feedback>

Read your document at .plan/<document-name> and revise it based on the feedback above.
Update the Summary section with what changed.
Set status to "complete" when done.
```

4. After revision, re-checkpoint (present to user again)
5. If revision count reaches `max_revisions` from the workflow (default 3), escalate:

```
This document has been revised <N> times. The agent may not be able to address
the feedback effectively.

A) Try one more revision with additional guidance
B) Accept as-is and move on
C) Abort the build
```

**If the user chooses C (Skip):** Mark document `status: complete` with a note that it was skipped, continue.

**If the user chooses D (Abort):** Stop execution. State what was completed and what remains.

### 3f. Parallel Execution

If `parallel_groups` in the workflow lists phases that can run concurrently:

1. Check that ALL dependencies for ALL phases in the group are met
2. If yes, invoke all agents in the group simultaneously using multiple Agent tool calls in a single message
3. Wait for all to complete
4. Checkpoint each one that requires it (sequentially, so the user reviews one at a time)

---

## Step 4: Update PLAN.md Progress

After each phase completes (or is skipped), update PLAN.md:

Change `- [ ] Phase N: <name>` to `- [x] Phase N: <name>` for completed phases.

Also update the PLAN.md frontmatter `status`:
- `in-progress` while phases remain
- `complete` when all phases are done

---

## Step 5: Final Summary

When all phases are complete:

```bash
git diff --stat 2>/dev/null || true
```

Output:

```
BUILD COMPLETE
══════════════════════════════════════
Task:     <task>
Phases:   <completed>/<total> complete
Revisions: <total revision count across all phases>

Documents:
  ✓ <document-name> — <one-line summary from each>
  ...

Files changed:
  <git diff --stat output>

Next steps:
  - /review — pre-landing code review
  - /qa — test the implementation
  - /ship — create PR and ship
══════════════════════════════════════
```

---

## Resume Logic

All state is in `.plan/` document frontmatter. If a session is interrupted:

1. `/build` re-reads all frontmatter (Step 2)
2. Skips phases with `status: complete`
3. For `status: in-progress` with a Summary section → marks complete, moves on
4. For `status: in-progress` without Summary → re-invokes the agent
5. For `status: revision-needed` → re-invokes with previous revision feedback
6. Resumes from the first incomplete phase

This means `/build` is fully idempotent — safe to run multiple times.

---

## Important Rules

- **You are the orchestrator, not an implementer.** Never write code yourself. Always delegate to agents.
- **One agent per phase.** Each phase invokes exactly one agent that owns one document.
- **Documents are truth.** Frontmatter status is the single source of truth for progress.
- **Checkpoints are mandatory.** If the workflow says checkpoint, you must present to the user. Do not skip.
- **Revisions are bounded.** Default max 3. Escalate after that.
- **Parallel when possible.** Use parallel agent invocation for independent phases.
- **Resume is automatic.** Just run `/build` again — it picks up where it left off.
- **Abort is clean.** If the user aborts, all completed work remains in `.plan/` documents.
