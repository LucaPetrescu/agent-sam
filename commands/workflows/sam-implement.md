---
name: workflows:sam-implement
description: Implementation-only workflow. Takes a plan file produced by /workflows:sam-plan (or a direct task description) and implements it using the software engineer agent in a review loop. No planning is done here.
model: opus
argument-hint: [plan-file-path] | [task-description]
allowed-tools:
  - Skill
  - Bash(ls:*)
  - Bash(find:*)
  - Bash(cat:*)
  - Bash(head:*)
  - Bash(tail:*)
  - Bash(grep:*)
  - Bash(pwd:*)
  - Bash(date:*)
  - Bash(git log:*)
  - Bash(git status:*)
  - Bash(git diff:*)
  - Bash(npm test:*)
  - Bash(npm run test:*)
  - Bash(npm run build:*)
  - Bash(npm run lint:*)
  - Bash(npm run format:*)
  - Bash(npm run typecheck:*)
  - Bash(go test:*)
  - Bash(go build:*)
  - Bash(go vet:*)
  - Bash(golangci-lint:*)
  - Bash(pytest:*)
  - Bash(cargo test:*)
# Ref: https://code.claude.com/docs/en/slash-commands#frontmatter
---

# Implement Workflow

You are implementing a plan. No architecture or design work happens here — only code. Run autonomously until all acceptance criteria are satisfied or escalation is required.

## Input

$ARGUMENTS

---

## Phase 0: Resolve Input

Determine the type of input:

1. **Plan file** — If the input is a file path ending in `.md` (e.g. `plans/2025-04-05-my-feature.md`), read the file and extract:
   - The implementation plan (phased steps)
   - The tech stack and conventions
   - The UI design system and any produced screens (if present)
   - The acceptance criteria

2. **Task description** — If the input is free text, use it directly as the implementation task. Infer acceptance criteria from the description if none are explicitly stated.

---

## Phase 1: Implementation

Spawn the **Software Engineer Agent** (`@agent:sam-software-engineer`) with:

- The full implementation plan and phased steps
- The tech stack and project conventions
- The acceptance criteria
- The UI design system and component files (if the plan includes a UI section) — pass these directly so the engineer integrates them without reimplementing

The engineer will implement each phase of the plan, run available validation commands (lint, format, test, build), and report a completion summary.

**Wait for the engineer to complete before proceeding.**

---

## Phase 2: Code Review

Once the engineer reports `COMPLETE`, spawn the **Code Reviewer Agent** (`@agent:sam-code-reviewer`) with:

- The acceptance criteria
- The list of files created or modified (from the engineer's completion report)
- The task description and conventions for context

**If the engineer reports `BLOCKED`:**

- Present the blockers to the user immediately with what was tried
- Ask how to proceed before continuing

**Wait for the code reviewer to complete before proceeding.**

---

## Phase 3: Iteration Loop

Evaluate the code reviewer's report:

**If `APPROVED`** — proceed to Phase 4.

**If `CHANGES_REQUIRED`:**

1. Pass the full review report (blockers + failed acceptance criteria) back to the **Software Engineer Agent**
2. The engineer addresses the blockers and failing criteria
3. Spawn the **Code Reviewer Agent** again with the same acceptance criteria
4. Repeat until `APPROVED` or the iteration limit is reached

**Maximum 3 iterations total.** After 3 loops without `APPROVED`:

Stop and escalate to the user:

- Which blockers and criteria are still failing
- What was attempted in each iteration
- Concrete options for how to proceed

Do not continue iterating beyond 3 times without user approval.

---

## Phase 4: Completion

When all acceptance criteria are satisfied, report:

```markdown
## Implementation Complete

### What was implemented

[Summary of the changes made]

### Files created or modified

[List with brief description of each]

### Acceptance criteria

- [x] Criterion 1 — how it was satisfied
- [x] Criterion 2 — how it was satisfied

### Validation

[Results of lint / format / test / build commands that were run]

### Next steps

[Any follow-up work, deployment steps, or open concerns noted during implementation]
```

---

## Rules

- **Implementation only** — do not re-plan, re-architect, or re-design; if the plan is incomplete or contradictory, ask the user before guessing
- **Max 3 iterations** — escalate to the user after 3 loops without passing all acceptance criteria
- **Pass context forward** — the full plan, conventions, and design system must be passed to the engineer in one go; never make the agent rediscover what the plan already contains
- **Be transparent about phase transitions** — announce clearly when moving between phases
