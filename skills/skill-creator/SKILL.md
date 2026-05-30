---
name: skill-creator
description: Guide for creating effective agent skills. Use this skill when creating a new skill or updating an existing one that extends agent capabilities with specialised knowledge, workflows, or tool integrations.
---

# Skill Creator

This skill provides guidance for creating effective skills for AI agents.

## About Skills

Skills are modular, self-contained packages that extend an agent's capabilities by providing specialised knowledge, workflows, and tools. They transform a general-purpose agent into a domain expert equipped with procedural knowledge.

### What Skills Provide

1. **Specialised workflows** — Multi-step procedures for specific domains
2. **Tool integrations** — Instructions for working with specific CLIs, APIs, or file formats
3. **Domain expertise** — Project-specific knowledge, schemas, business logic
4. **Bundled resources** — Scripts, references, and assets for complex or repetitive tasks

## Core Principles

### Concise is Key

The context window is shared with everything else: the system prompt, conversation history, other skill metadata, and the actual user request.

**Default assumption: the agent is already very smart.** Only add context it doesn't already have. Challenge each piece: "Does the agent really need this explanation?" and "Does this paragraph justify its token cost?"

Prefer concise examples over verbose explanations.

### Set Appropriate Degrees of Freedom

Match specificity to the task's fragility and variability:

- **High freedom (text instructions)** — Multiple approaches are valid; decisions depend on context
- **Medium freedom (pseudocode or parameterised scripts)** — A preferred pattern exists but some variation is acceptable
- **Low freedom (specific scripts, few parameters)** — Operations are fragile; a specific sequence must be followed exactly

### Anatomy of a Skill

Every skill consists of a required `SKILL.md` and optional bundled resources:

```
skill-name/
├── SKILL.md              (required)
│   ├── YAML frontmatter  (required) — name + description
│   └── Markdown body     (required) — instructions and guidance
└── references/           (optional) — docs loaded into context as needed
└── scripts/              (optional) — executable code for deterministic tasks
└── assets/               (optional) — files used in output (templates, icons, etc.)
```

#### SKILL.md frontmatter

Only two fields are required:

- `name` — the skill name (kebab-case)
- `description` — **the primary trigger**. Must describe both what the skill does AND when to use it. All "when to use" information belongs here — the body is only loaded after triggering.

Do not add any other frontmatter fields.

#### Bundled resources

**`references/`** — Documentation loaded into context as needed. Use for schemas, API docs, domain knowledge, detailed workflow guides. Keep SKILL.md lean by moving detailed material here.

**`scripts/`** — Executable code for tasks that are repeatedly rewritten or require deterministic reliability. Scripts may be executed without loading into context.

**`assets/`** — Files used in output but not loaded into context: templates, icons, boilerplate code, fonts.

#### What NOT to include

Do not create extraneous files: `README.md`, `CHANGELOG.md`, `INSTALLATION_GUIDE.md`, etc. The skill should only contain what an agent needs to do the job — not documentation about the skill itself.

## Progressive Disclosure

Skills use a three-level loading system:

1. **Metadata (name + description)** — Always in context (~100 words)
2. **SKILL.md body** — Loaded when the skill triggers (<500 lines recommended)
3. **Bundled resources** — Loaded only when Claude determines they're needed

Keep the SKILL.md body to essentials. When approaching 500 lines, split content into `references/` files and link to them explicitly from SKILL.md with clear guidance on when to read each one.

**Avoid deeply nested references** — all reference files should link directly from SKILL.md, one level deep.

### Patterns for splitting content

**Pattern 1 — High-level guide with references**

```markdown
## Advanced features

- For form filling: see [references/forms.md](references/forms.md)
- For API reference: see [references/api.md](references/api.md)
```

**Pattern 2 — Domain-specific organisation**

Organise by domain so only relevant content is loaded:

```
bigquery-skill/
├── SKILL.md
└── references/
    ├── finance.md
    ├── sales.md
    └── product.md
```

**Pattern 3 — Variant-specific details**

```
cloud-deploy/
├── SKILL.md        (workflow + provider selection)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

## Skill Creation Process

### Step 1: Understand the skill with concrete examples

Clarify how the skill will be used before writing anything. Ask questions like:

- "What functionality should this skill support?"
- "Can you give me examples of how this skill would be used?"
- "What would a user say that should trigger this skill?"

Conclude when you have a clear sense of the functionality and trigger conditions.

### Step 2: Plan the reusable contents

For each concrete usage example, identify:

1. What would need to be done from scratch each time
2. What scripts, references, or assets would eliminate that repetition

Examples:

- Repeated code → `scripts/`
- Schema or API docs re-discovered each time → `references/schema.md`
- Boilerplate project files → `assets/template/`

### Step 3: Create the skill directory

Create the skill folder under `skills/` in the project:

```bash
mkdir -p skills/<skill-name>
# Add subdirectories only as needed:
mkdir -p skills/<skill-name>/references
mkdir -p skills/<skill-name>/scripts
mkdir -p skills/<skill-name>/assets
```

### Step 4: Implement bundled resources

Create the `references/`, `scripts/`, and `assets/` files identified in Step 2.

- **Test all scripts** by running them — do not ship untested code
- **Delete any empty directories** that aren't needed

### Step 5: Write SKILL.md

1. Write the frontmatter with `name` and `description`
   - Description must cover both what the skill does and when to use it
   - Be specific about triggers — vague descriptions lead to the skill not firing
2. Write the body with concise, imperative instructions
   - Reference bundled resources explicitly with clear guidance on when to read them
   - Prefer examples over prose

### Step 6: Iterate

After using the skill on real tasks:

1. Notice where the agent struggles or makes wrong decisions
2. Identify whether the issue is in the description (triggering), the body (instructions), or missing resources
3. Update and test again

The description is the most common failure point — if the skill isn't triggering when expected, rewrite the description to be more specific about the context and triggers.
