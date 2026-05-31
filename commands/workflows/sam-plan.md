---
name: workflows:sam-plan
description: Planning-only workflow. Given a project idea or task description, runs architecture planning through a structured phase process, gets user approval, and saves a plan file to disk. No code is written. Run /workflows:sam-implement with the saved plan file to execute it.
model: opus
argument-hint: [project-idea] | [task-description]
allowed-tools:
  - Skill
  - Bash(ls:*)
  - Bash(find:*)
  - Bash(cat:*)
  - Bash(head:*)
  - Bash(grep:*)
  - Bash(pwd:*)
  - Bash(date:*)
  - Bash(git log:*)
  - Bash(git status:*)
# Ref: https://code.claude.com/docs/en/slash-commands#frontmatter
---

# Plan Workflow

You are producing a complete, approved plan for a project. **You will not write any code.** Your job is to run architecture planning, get the user's sign-off, and save the result to disk so it can be handed to `/workflows:sam-implement`.

## Input

$ARGUMENTS

---

## Phase 1: Architecture Planning

Spawn the **Software Architect Agent** (`@agent:sam-software-architect`) with the project idea or task description as input.

The architect will:

1. Interview the user about requirements, constraints, and goals
2. Define the system architecture and component breakdown
3. Select the full tech stack with rationale — including the UI/frontend stack if the project has a user-facing interface
4. Produce a UI plan (screen inventory, component breakdown, design direction) if the project has a UI and the user requests it
5. Establish best practices and conventions for the project
6. Produce a phased implementation plan with acceptance criteria
7. Export an architecture diagram (`.mmd` + `.png`) via the `diagram-builder` skill

**Wait for the architect to complete before proceeding.**

Collect from the architect's output:

- The full architecture plan (including UI plan if produced)
- Tech stack decisions (including UI stack if applicable)
- Acceptance criteria
- Whether the project includes a user-facing UI

Once the architect finishes, notify the user:

> **Architecture diagram exported**
>
> - `architecture-diagram.mmd` — Mermaid source
> - `architecture-diagram.png` — rendered image
>
> Both files are in the "DOCUMENTATION" folder located in the root of the project. If no such folder exists, create it in the root of the project

---

## Phase 2: UI Design (only if explicitly requested)

**Do not spawn the UI designer automatically.**

If the project has a user-facing UI and the user has not already explicitly requested detailed UI design, ask:

> "The architect has produced a UI plan covering screens, components, and design direction. Would you also like full UI design — a complete design system with implementation-ready HTML/CSS screens? This requires the UI Designer agent."

- If the user says **yes** → spawn the **UI Designer Agent** (`@agent:sam-ui-designer`) with:
  - The project description
  - The UI tech stack and UI plan from the architect
  - Any brand or visual constraints the user mentioned

  Wait for the designer to complete before proceeding.

- If the user says **no** → skip this phase entirely.

---

## Phase 3: Plan Approval

Present a consolidated summary to the user:

```
## Plan Summary

### Architecture
[Key decisions: system style, tech stack, major components]

### UI Plan
[Screen inventory, component breakdown, design direction — or "Not included" if skipped]

### UI Design System
[Design system summary and screens produced — or "Not requested" if skipped]

### Implementation Plan
[The phased breakdown from the architect]

### Acceptance Criteria
[Full list from the architect]
```

Ask the user:

1. Does the architecture plan look correct and complete?
2. Does the UI plan / design match the vision? (if applicable)
3. Are there any changes to make before saving?
4. What should this plan be named? (suggest a kebab-case name based on the project)

If the user requests changes:

- Route architecture feedback back to the architect agent
- Route UI/design feedback back to the designer agent (if UI design was done)
- Re-present the updated summary until the user approves

**Do not proceed to Phase 4 until the user explicitly approves.**

---

## Phase 4: Save Plan to Disk

Once approved, save the complete plan to the `DOCUMENTATION/plans/` directory:

- Filename format: `DOCUMENTATION/plans/YYYY-MM-DD-<feature-name>.md`
- Example: `DOCUMENTATION/plans/2025-04-05-task-management-app.md`

Use this structure:

```markdown
# Plan: [Feature Name]

**Created:** YYYY-MM-DD
**Status:** Approved

## Overview

[2-3 sentence summary of what is being built]

## Architecture

[Full architecture plan from the Software Architect Agent]

## UI Plan

[Screen inventory, component breakdown, design direction — or "Not included" if skipped]

## UI Design System

[Design system summary and list of screens produced — or "Not requested" if skipped]

## Acceptance Criteria

[Full acceptance criteria list]

## Notes

[Any decisions, trade-offs, or constraints discussed during the approval process]
```

Confirm to the user:

> **Plan saved:** `DOCUMENTATION/plans/YYYY-MM-DD-<feature-name>.md`
>
> Run the following to implement it:
>
> ```
> /workflows:sam-implement DOCUMENTATION/plans/YYYY-MM-DD-<feature-name>.md
> ```

---

## Rules

- **No code, no file modifications** — this workflow is read-only except for writing the plan file
- **UI plan is the architect's job** — the architect always produces a UI plan (screens, components, design direction) when the project has a UI; this does not require the UI Designer agent
- **UI Designer is opt-in** — never spawn `@agent:sam-ui-designer` unless the user explicitly asks for it
- **Pass context forward** — each agent's output is passed as input to the next; never make the user repeat themselves
- **Be transparent about phase transitions** — announce clearly when moving from one phase to the next
