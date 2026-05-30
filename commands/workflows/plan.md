---
name: workflows-sam:plan
description: Planning-only workflow. Given a project idea or task description, runs architecture planning and UI design through a structured phase process, gets user approval, and saves a plan file to disk. No code is written. Run /workflows-sam:implement with the saved plan file to execute it.
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

You are producing a complete, approved plan for a project. **You will not write any code.** Your job is to run architecture and UI design, get the user's sign-off, and save the result to disk so it can be handed to `/workflows:implement`.

## Input

$ARGUMENTS

---

## Phase 1: Architecture Planning

Spawn the **Software Architect Agent** (`@agent:software-architect`) with the project idea or task description as input.

The architect will:

1. Interview the user about requirements, constraints, and goals
2. Define the system architecture and component breakdown
3. Select the tech stack with rationale
4. Establish best practices and conventions for the project
5. Produce a phased implementation plan with acceptance criteria
6. Export an architecture diagram (`.mmd` + `.png`) via the `diagram-builder` skill

**Wait for the architect to complete before proceeding.**

Collect from the architect's output:

- The full architecture plan
- Tech stack decisions
- Acceptance criteria
- Whether the project includes a user-facing UI (frontend, web app, mobile)

Once the architect finishes, notify the user:

> **Architecture diagram exported**
>
> - `architecture-diagram.mmd` — Mermaid source
> - `architecture-diagram.png` — rendered image
>
> Both files are in the current working directory.

---

## Phase 2: UI Design (conditional)

**Skip this phase if the project has no user-facing UI.**

If the architecture plan includes a frontend, web app, or any user-facing interface, spawn the **UI Designer Agent** (`@agent:ui-designer`) with:

- The project description
- The tech stack chosen in Phase 1 (framework, any existing brand constraints)
- Any relevant context from the architecture plan (what screens exist, what data they show)

The designer will:

1. Interview the user about brand, personality, and visual direction
2. Define the design system (colours, typography, spacing, component tokens)
3. Produce full, implementation-ready HTML/CSS or component screens
4. Provide a developer handoff summary

**Wait for the designer to complete before proceeding.**

---

## Phase 3: Plan Approval

Present a consolidated summary to the user:

```
## Plan Summary

### Architecture
[Key decisions: system style, tech stack, major components]

### UI Design
[Design system summary and screens produced — or "No UI" if skipped]

### Implementation Plan
[The phased breakdown from the architect]

### Acceptance Criteria
[Full list from the architect]
```

Ask the user:

1. Does the architecture plan look correct and complete?
2. Does the UI design match the vision? (if applicable)
3. Are there any changes to make before saving?
4. What should this plan be named? (suggest a kebab-case name based on the project)

If the user requests changes:

- Route architecture feedback back to the architect agent
- Route UI/design feedback back to the designer agent
- Re-present the updated summary until the user approves

**Do not proceed to Phase 4 until the user explicitly approves.**

---

## Phase 4: Save Plan to Disk

Once approved, save the complete plan to the `plans/` directory:

- Filename format: `plans/YYYY-MM-DD-<feature-name>.md`
- Example: `plans/2025-04-05-task-management-app.md`

Use this structure:

```markdown
# Plan: [Feature Name]

**Created:** YYYY-MM-DD
**Status:** Approved

## Overview

[2-3 sentence summary of what is being built]

## Architecture

[Full architecture plan from the Software Architect Agent]

## UI Design

[Design system summary and list of screens produced — or "No UI component" if skipped]

## Acceptance Criteria

[Full acceptance criteria list]

## Notes

[Any decisions, trade-offs, or constraints discussed during the approval process]
```

Confirm to the user:

> **Plan saved:** `plans/YYYY-MM-DD-<feature-name>.md`
>
> Run the following to implement it:
>
> ```
> /workflows-sam:implement plans/YYYY-MM-DD-<feature-name>.md
> ```

---

## Rules

- **No code, no file modifications** — this workflow is read-only except for writing the plan file
- **UI is conditional** — only invoke the UI Designer if the project includes a user-facing interface
- **Pass context forward** — each agent's output is passed as input to the next; never make the user repeat themselves
- **Be transparent about phase transitions** — announce clearly when moving from one phase to the next
