---
name: read-project-context
description: Load project context from a persisted memory store (e.g. Obsidian vault) into the current conversation so Claude can make more informed decisions and code changes.
use_when: User wants to load project context at the start of a conversation, or before working on a feature that benefits from domain knowledge.
user-invocable: true
allowed-tools: Read, Glob, Grep, AskUserQuestion
---

# Read Project Context Skill

Load persisted project knowledge into the current conversation so subsequent work is informed by business logic, architectural decisions, integration details, and domain constraints.

## Arguments

`$ARGUMENTS` should contain the project name, e.g. `/read-project-context my-project`.

Optionally, the user can append a topic filter: `/read-project-context my-project authentication` to load only context files relevant to "authentication".

Parse `$ARGUMENTS` as:

- First word = project name
- Remaining words (if any) = topic filter

If no project name is provided, ask the user which project to load context for.

---

## Phase 1: Resolve the project memory location

### 1.1 Check auto memory for saved location

Look in the user's auto memory directory for a file named `reference_project_context_{project_name}.md`. This contains the base folder path where project context is stored.

The auto memory directory is determined by the current working directory's project memory path. Check `MEMORY.md` there for existing project context references.

### 1.2 No saved location

If no saved location exists for this project:

1. Ask the user: **"I don't have a saved context location for `{project_name}`. Where is the project context folder? (e.g. `/Users/you/vault/Projects/my-project/`)"**
2. Once provided, save a `reference` memory so future invocations find it automatically:
   - File: `reference_project_context_{project_name}.md`
   - Add entry to `MEMORY.md`

### 1.3 Validate

Verify the folder exists and contains an `index.md`. If the folder exists but has no `index.md`, warn the user: "The project context folder exists but has no index. Run `/project-context {project_name}` first to populate it."

---

## Phase 2: Read the index

Read `index.md` from the project memory folder. This is the map of all context files organized by category.

---

## Phase 3: Determine what to load

### 3a: Topic filter provided

If the user specified a topic filter (e.g. "authentication", "provider integration", "scheduling"):

1. Scan the index for entries whose **title**, **filename**, or **description** match the topic (case-insensitive, partial match is fine)
2. Read only those matching context files
3. If no matches found, tell the user: "No context files matched '{topic}'. Here are the available topics:" and list the index entries so they can pick.

### 3b: No topic filter — load everything

Read **all** context files listed in the index. If the project has many files (>10), read them in batches to avoid overwhelming context, prioritizing by:

1. Files in the **Constraints** and **Business Logic** categories first (these most commonly affect correctness)
2. Then **Architecture** and **Data Model**
3. Then **Integrations** and **Process**

---

## Phase 4: Present loaded context

After reading, output a structured summary so the user (and you) can confirm what's loaded:

```
Loaded project context for {project_name}:

**{N} context files loaded:**

**Business Logic**
- {Topic title} - {N} key facts
- {Topic title} - {N} key facts

**Architecture**
- {Topic title} - {N} key facts

**Constraints**
- {Topic title} - {N} key facts

{If topic-filtered: "Filtered to: {topic}. Run `/read-project-context {project_name}` without a filter to load all."}
```

Keep the summary concise — list the file names and fact counts, not the full content. The full content is already in your working memory from the reads.

---

## Phase 5: Internalize for the session

After loading, you now have domain knowledge that should inform your work for the rest of this conversation. Apply it by:

- **When writing code:** Check loaded constraints and business logic before making assumptions
- **When reviewing:** Flag violations of documented invariants
- **When designing:** Respect architectural decisions unless the user explicitly wants to change them
- **When asking questions:** Don't ask about things already covered in loaded context

You do NOT need to quote the context back verbatim when using it. Just apply the knowledge naturally, as if you already knew it.

---

## Edge Cases

**Empty project context:** If index.md exists but has no entries, tell the user: "Project context for `{project_name}` is empty. Work through a session and run `/project-context {project_name}` to start building it up."

**Stale context warning:** If `log.md` exists, check the most recent date entry. If it's older than 30 days, add a note: "Last context update was {date}. Some facts may be outdated — verify against current code when in doubt."

**Large project context (>15 files):** After loading, suggest the user use topic filters in future invocations for faster loading: "This project has {N} context files. Consider using `/read-project-context {project_name} {topic}` to load specific areas."
