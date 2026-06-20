---
name: update-project-context
description: Extract context from the current session and persist it to a project memory store (e.g. Obsidian vault). Maintains an index, individual context files, and a change log.
use_when: User wants to save project context, capture business logic, record architectural decisions, or update project knowledge from the current conversation.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# Project Context Skill

Extract facts, business logic, and decisions from the current conversation and persist them to a structured project memory store.

## Arguments

`$ARGUMENTS` should contain the project name, e.g. `/update-project-context my-project`.

If no project name is provided, ask the user which project this context belongs to.

---

## Phase 1: Resolve the project memory location

### 1.1 Check auto memory for saved location

Look in the user's auto memory directory for a file that maps this project name to a folder path. The memory file will be named `reference_project_context_{project_name}.md` and contain the base folder path.

The auto memory directory is determined by the current working directory's project memory path. Check `MEMORY.md` there for existing project context references.

### 1.2 First-time setup

If no saved location exists for this project:

1. Ask the user: **"Where should I save project context for `{project_name}`? Provide the full path to the folder (e.g. an Obsidian vault subfolder like `/Users/you/vault/Projects/my-project/`)."**
2. Once the user provides a path, save a `reference` memory:
   - File: `reference_project_context_{project_name}.md`
   - Content: the folder path
   - Add entry to `MEMORY.md`
3. Create the folder if it doesn't exist.
4. Create the initial structure:
   - `index.md` - empty index (see format below)
   - `log.md` - empty changelog

### 1.3 Validate existing location

If a saved location exists, verify the folder still exists. If it doesn't, inform the user and ask for a new path.

---

## Phase 2: Extract context from the current session

Scan the full conversation history and identify **facts worth persisting**. These include:

- **Business logic / domain rules** - e.g. "discounts only apply after account verification"
- **Architectural decisions** - e.g. "we chose X over Y because Z"
- **System behavior** - e.g. "the sync job runs nightly via a background worker"
- **Integration details** - e.g. "the provider sends webhooks to /api/v1/events"
- **Data model insights** - e.g. "users can belong to multiple groups"
- **Constraints / invariants** - e.g. "tenant_id must always be checked for authorization"
- **Process / workflow knowledge** - e.g. "releases are cut every Thursday"

**Do NOT extract:**

- Ephemeral debugging context (stack traces, one-off errors)
- Code snippets (the code itself is the source of truth)
- Task-specific instructions that don't generalize
- Anything already documented in CLAUDE.md or AGENTS.md

For each extracted fact, note:

- The **fact** itself (concise, declarative sentence)
- The **category** it belongs to (business-logic, architecture, integration, data-model, process, constraint)
- Any **source context** (e.g. "discussed while implementing PROJ-1234")

---

## Phase 3: Load the project index

Read `index.md` from the project memory folder. The index has this format:

```markdown
# Project Context Index

Quick-reference map of where project knowledge lives. Each entry links to a context file.

## Business Logic

- [Discount eligibility rules](discount-eligibility-rules.md) - When and how discounts can be applied
- [Billing calculation](billing-calculation.md) - Daily billing computation logic

## Architecture

- [Task queue design](task-queue-design.md) - Background task patterns and concurrency strategies

## Integrations

- [Provider webhook handling](provider-webhook-handling.md) - Inbound event processing from external providers

## Data Model

- [User group membership](user-group-membership.md) - Group membership rules

## Process

- [Release workflow](release-workflow.md) - Release cadence and freeze windows

## Constraints

- [Authorization invariants](authorization-invariants.md) - Tenant scoping rules
```

Each section groups related context files by category. Each entry is a single line with a link and a short description.

---

## Phase 4: Reconcile facts against existing context

For each extracted fact:

### 4.1 Search the index for a matching context file

Look at the index entries and determine if this fact belongs in an **existing** context file. Read the relevant file(s) to check.

### 4.2 Classify the update type

| Situation                                                              | Action                                       |
| ---------------------------------------------------------------------- | -------------------------------------------- |
| **New fact, no existing file covers this topic**                       | Create a new context file (Step 5a)          |
| **New fact, existing file covers this topic but doesn't mention this** | Append to existing file (Step 5b)            |
| **Updated fact, consistent with existing**                             | Update in place (Step 5b)                    |
| **Updated fact, CONTRADICTS existing**                                 | **Ask the user for clarification** (Step 5c) |

### 4.3 Contradiction detection

A contradiction exists when:

- An existing fact states X, but the new context implies NOT X or a different value
- An existing decision says "we chose A", but the conversation indicates a switch to B
- A numeric value, date, or status has changed (e.g. "releases on Thursday" vs "releases on Tuesday")

**When a contradiction is found:**

1. Show the user both the existing fact and the new fact
2. Ask: "These seem to conflict. Which is correct, or has this changed?"
3. Only update after the user confirms

---

## Phase 5: Write updates

### 5a: Create a new context file

**File naming convention:**

- Use lowercase kebab-case
- Be descriptive but concise (3-5 words)
- Name should describe the _topic_, not the _event_ that surfaced it
- Good: `eligibility-rules.md`, `webhook-handling.md`
- Bad: `ticket-4077-fix.md`, `session-2024-01-15.md`, `misc-notes.md`

**File format:**

```markdown
# {Title}

{Brief 1-2 sentence summary of what this context file covers.}

## Key Facts

- {Fact 1}
- {Fact 2}

## Details

{Longer explanation if needed. Use subsections as the topic grows.}

## Sources

- {Where this knowledge came from, e.g. "Discussed during PROJ-1234 implementation, 2026-04-11"}
```

### 5b: Update an existing context file

- Read the file
- Add the new fact in the appropriate section
- If updating an existing bullet, edit in place
- Preserve the existing structure and voice
- Add a new source entry if the update came from a different context

### 5c: Handle contradictions

After user confirms the correct version:

- Update the fact in the context file
- Add a note in the Sources section: `"Updated {YYYY-MM-DD}: {old value} -> {new value} (confirmed by user)"`

---

## Phase 6: Update the index

After all context files are written/updated:

1. Read the current `index.md`
2. Add entries for any **new** context files, placed under the correct category section
3. If a new category is needed, add a new `##` section
4. Keep entries sorted alphabetically within each section
5. Ensure every context file in the folder has an index entry

---

## Phase 7: Update the changelog

Append to `log.md` in **reverse chronological order** (newest at top, just below the heading).

**Format:**

```markdown
# Project Context Changelog

## {YYYY-MM-DD}

- Created [some-topic.md] - Added initial facts about X
- Updated [authorization-invariants.md] - Added tenant scoping for new endpoint

## {YYYY-MM-DD}

- Created [billing-calculation.md] - Initial documentation of billing logic
```

Rules:

- One `## YYYY-MM-DD` heading per day (reuse if same day)
- Each change is a bullet: `Created` or `Updated` + filename + brief description of what changed
- Reverse chronological: newest day at top, newest entry first within the day

---

## Phase 8: Confirm with user

Present a summary:

```
Project context updated for {project_name}:

**New files:**
- {filename}.md - {N} facts added

**Updated files:**
- {filename}.md - {N} facts added

**Changelog:** log.md updated

**Contradictions resolved:** {count} (or list them)
```

---

## Quality Guidelines

**Good context entries:**

- "Discounts can only be applied after enrollment is complete and the user has a verified payment method"
- "We use mutex locks on background tasks to prevent duplicate processing per entity"
- "Provider webhook payloads include user_id but NOT tenant_id — we look it up from our mapping table"

**Bad context entries:**

- "Fixed a bug" (too vague, no lasting knowledge)
- "The function is at line 42 of handler.py" (will go stale)
- "Someone asked me to add this endpoint" (ephemeral, not knowledge)

**File naming — think about the future reader:**
The index is how a future agent (or human) finds relevant context. Names should be self-explanatory without needing to open the file. Prefer domain terms over implementation terms:

- `schedule-sync-rules.md` over `cron-job-config.md`
- `user-onboarding-flow.md` over `post-endpoint-logic.md`
