---
name: team
version: 1.0.0
description: |
  Dynamic team builder for software development. Analyzes the project and task,
  selects specialist agents from a role catalog (or creates custom ones), writes
  agent definitions to .claude/agents/, and produces a structured plan with
  document hierarchy in .plan/. Use when asked to "build a team", "assemble agents",
  "plan this feature", or "/team <task>".
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
echo '{"skill":"team","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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

# Dynamic Team Builder

You are running the `/team` workflow. Analyze the project and task, select the
right specialist agents, write agent definitions and a structured plan.

**Input:** `$ARGUMENTS` contains the task description (e.g., "add OAuth2 authentication").

---

## Step 1: Gather Project Context

Read all of these in parallel to understand the project:

```bash
cat CLAUDE.md 2>/dev/null || true
cat README.md 2>/dev/null || true
```

```bash
# Detect package manager and tech stack
cat package.json 2>/dev/null | head -80 || true
cat Cargo.toml 2>/dev/null | head -40 || true
cat go.mod 2>/dev/null | head -20 || true
cat Gemfile 2>/dev/null | head -40 || true
cat requirements.txt 2>/dev/null | head -40 || true
cat pyproject.toml 2>/dev/null | head -40 || true
```

```bash
# Directory structure (depth 2)
find . -maxdepth 2 -type d \
  -not -path '*/node_modules/*' \
  -not -path '*/.git/*' \
  -not -path '*/dist/*' \
  -not -path '*/build/*' \
  -not -path '*/__pycache__/*' \
  -not -path '*/target/*' \
  -not -path '*/.next/*' | sort | head -60
```

```bash
# Config files hint at stack
ls -1 tsconfig*.json vite.config.* tailwind.config.* next.config.* \
  webpack.config.* docker-compose.* Dockerfile .eslintrc* .prettierrc* \
  jest.config.* vitest.config.* playwright.config.* 2>/dev/null || true
```

```bash
# Existing agents and plan state
ls .claude/agents/ 2>/dev/null || echo "NO_AGENTS"
ls .plan/ 2>/dev/null || echo "NO_PLAN"
```

```bash
# Recent git activity
git log --oneline -10 2>/dev/null || true
git diff --stat HEAD~5..HEAD 2>/dev/null | tail -5 || true
```

From these outputs, extract and note:
- **Tech stack:** languages, frameworks, databases, tools
- **Project type:** web app, API, CLI, library, data pipeline, mobile, etc.
- **Has backend:** true/false (look for server dirs, API routes, models, db config)
- **Has frontend:** true/false (look for components/, pages/, app/, views/, CSS/Tailwind)
- **Conventions:** test framework, build tool, commit style, directory patterns
- **Current state:** existing agents, existing plan, recent work

---

## Step 2: Classify the Task

Analyze `$ARGUMENTS` to determine:

**Task type** (pick one):
- `feature` — new functionality
- `bugfix` — fixing broken behavior
- `refactor` — restructuring without behavior change
- `migration` — data or schema migration
- `optimization` — performance improvement
- `docs` — documentation only
- `devops` — deployment or infrastructure
- `config` — configuration changes only

**Scope** (estimate from task description + project size):
- `small` — 1-3 files, localized change
- `medium` — 4-10 files, crosses a few modules
- `large` — 10+ files, multiple subsystems

**Stack layers** touched:
- `backend` / `frontend` / `fullstack` / `data` / `infrastructure`

---

## Step 3: Select Roles from Catalog

Read the role catalog:

```bash
cat "${CLAUDE_SKILL_DIR}/catalog.yaml"
```

Evaluate each role's `trigger_conditions` against the project analysis and task classification:

1. **Check tech stack:** Does `tech_stack_includes` match detected technologies?
2. **Check keywords:** Does `mentions` match words in `$ARGUMENTS`?
3. **Check project shape:** Does `has_backend`/`has_frontend` match?
4. **Check task type:** Does `task_type` include the classified type?
5. **Apply `except_when`:** Skip roles where exception conditions match.
6. **Apply `always_with`:** When a role is selected, auto-include its companions.
7. **Apply scope cap:** small tasks max 3 roles, medium max 5, large no cap.

**Selection priority (when hitting the cap):**
1. Implementation roles first (engineers who write code)
2. Architecture/design roles second (they guide the engineers)
3. Review roles last (security, performance — important but capped first)

**Consider custom roles** when:
- The project uses specialized technology not covered by catalog roles
- The task requires domain expertise beyond general software engineering
- Follow the `custom_roles.guidelines` in the catalog for creation rules

---

## Step 3.5: Check for Existing State

If Step 1 found existing agents or plan files:

1. Read `.plan/PLAN.md` and `.plan/workflow.yaml` if they exist
2. Compare the existing team and task against the new request

Use AskUserQuestion:
```
Found existing plan: "<existing task title>"
Team: <existing roles>
Status: <phases completed/total>

A) Update — modify the team for the new task, keep any completed work
B) Replace — archive existing plan to .plan/archive/<timestamp>/, start fresh
C) Resume — just run /build to continue the existing plan
```

If no existing state, skip to Step 4.

---

## Step 4: User Confirmation

Present the proposed team via AskUserQuestion:

```
Building team for: "<task from $ARGUMENTS>"
Project: <project name> (<tech stack summary>)

TEAM:
  1. <Role Name> (opus) — <one-line purpose>
  2. <Role Name> (opus) — <one-line purpose>
  ...

EXECUTION ORDER:
  Phase 1: <Role> → <document> (checkpoint)
  Phase 2: <Role> → <document> (checkpoint)
  Phase 3: <Role> → <document>
  ...

Parallel groups: <if any phases can run in parallel, note them>

RECOMMENDATION: Choose A — this team covers all aspects of the task.

A) Approve
B) Add a role — tell me which
C) Remove a role — tell me which
D) Start over with different task
```

If the user chooses B or C, adjust and re-present. If D, restart from Step 2.

---

## Step 5: Generate Agent Definitions

For each selected role, write a `.claude/agents/<role-id>.md` file.

```bash
mkdir -p .claude/agents
```

Each agent file follows this format:

```markdown
---
name: <role-id>
description: "<Role Name> — working on: <task summary>"
tools: [<tools from catalog>]
model: opus
skills: [project-context]
---

# <Role Name>

## Your Assignment
You are the <Role Name>. Your task:
> <task from $ARGUMENTS>

## Context
- Project: <project name> (<tech stack>)
- Branch: <current git branch>
- Your document: .plan/<document-name>
- Depends on: <list of upstream .plan/ documents this role must read first>

## Instructions
<prompt_skeleton from catalog, with project-specific paths resolved>

## Completion Protocol
When done:
1. Update your document's frontmatter: set `status: complete`, update `updated` timestamp
2. Add a ## Summary section at the TOP of your document (after frontmatter) with:
   - What you produced (1-3 bullet points)
   - Files created or modified (list each)
   - Any concerns or open questions
3. Your document is the deliverable — make it complete and self-contained
```

**Important:** Write each agent file with the prompt_skeleton customized for this specific
project and task. Replace generic references with actual paths, file names, and conventions
discovered in Step 1.

---

## Step 6: Generate Project Context Skill

Write a project context skill that every agent will have preloaded:

```bash
mkdir -p .claude/skills/project-context
```

Write `.claude/skills/project-context/SKILL.md`:

```markdown
---
name: project-context
description: "Project conventions for <project name>"
---
# Project: <project name>
## Tech Stack: <detected stack>
## Test command: <from CLAUDE.md or detected>
## Build command: <from CLAUDE.md or detected>
## Directory structure: <key directories and their purpose>
## Conventions: <naming, patterns, commit style>
## Branch: <current branch>
## Task: <task from $ARGUMENTS>
```

Keep this concise — under 30 lines. It's context, not documentation.

---

## Step 7: Generate Plan Documents

Create the `.plan/` directory and all documents:

```bash
mkdir -p .plan
```

**7a. Write workflow.yaml:**

```yaml
version: 1
task: "<task from $ARGUMENTS>"
created: "<today's date ISO8601>"
team: [<role-ids in execution order>]

phases:
  - id: <phase-id>
    agent: <role-id>
    document: <document-name>.md
    depends_on: [<upstream phase ids>]
    checkpoint: <true for architecture/requirements/test phases, false for implementation>
  ...

parallel_groups: [<lists of phase ids that can run concurrently>]

rejection_protocol:
  max_revisions: 3
  on_max: ask_user
```

**Checkpoint rules:**
- Requirements: always checkpoint (user validates scope)
- Architecture/design: always checkpoint (user validates approach)
- Implementation: no checkpoint (let it flow)
- Tests: always checkpoint (user validates coverage)
- Reviews (security, performance): always checkpoint (user must see findings)

**Parallel groups:** Identify phases with no dependency between them. Common:
- Backend implementation + Frontend implementation (after their respective architecture phases)
- Security review + Performance review (after implementation)

**7b. Write PLAN.md** (max 50 lines):

```markdown
---
title: "<task>"
status: pending
team: [<role-ids>]
created: <today>
---
# <Task Title>

## Objective
<1-2 sentences: what we're building and why>

## Team
| Role | Agent | Document |
|------|-------|----------|
| <Role Name> | @<role-id> | <document>.md |
...

## Progress
- [ ] Phase 1: <phase name>
- [ ] Phase 2: <phase name>
...
```

**7c. Write document templates:**

For each role's document, copy the corresponding template from the skill directory
and fill in the frontmatter:

```bash
cat "${CLAUDE_SKILL_DIR}/templates/<document-name>"
```

Copy each template to `.plan/<document-name>`, updating:
- `created`: today's date
- `owner`: the role-id
- `depends-on`: upstream documents from the workflow

---

## Step 8: Gitignore

Add `.plan/` to `.gitignore` unless the user has opted it in:

```bash
grep -q '\.plan/' .gitignore 2>/dev/null || echo '.plan/' >> .gitignore
```

---

## Step 9: Output Summary

```
TEAM BUILT
══════════════════════════════════════
Task:     <task>
Project:  <project name> (<stack>)
Team:     <N roles>
Phases:   <N phases> (<M checkpoints>)

Agents:   .claude/agents/<role-id>.md (×N)
Plan:     .plan/PLAN.md
Workflow: .plan/workflow.yaml
Context:  .claude/skills/project-context/SKILL.md

Next: Run /build to start execution.
══════════════════════════════════════
```

---

## Re-run Handling

If `/team` is run and `.plan/` already exists, Step 3.5 handles it.

When **updating** an existing team:
- Preserve completed documents (don't overwrite)
- Add/remove agent definitions as needed
- Update workflow.yaml to reflect new phases
- Update PLAN.md progress to reflect current state

When **replacing**:
```bash
ARCHIVE=".plan/archive/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$ARCHIVE"
mv .plan/*.md .plan/*.yaml "$ARCHIVE/" 2>/dev/null || true
```
Then proceed with fresh generation from Step 5.

---

## Important Rules

- **Every agent uses model: opus.** No model tiers — every agent gets best reasoning.
- **Reviewers are read-only.** Security and Performance reviewers get no Write/Edit tools.
- **Documents are the deliverables.** Every agent produces exactly one document in `.plan/`.
- **Prompt skeletons are customized.** Don't copy catalog prompts verbatim — resolve paths, inject project context.
- **Scope cap is firm.** Small tasks: max 3 roles. Don't over-staff simple changes.
- **The user confirms before anything is written.** Step 4 must happen before Steps 5-8.
- **`always_with` is automatic.** If Backend Architect is selected, Backend Engineer comes along — no need to ask.
