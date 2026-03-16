---
name: create-context
description: "Create and update structured context files for Claude to load in future sessions. Scans vault, asks guided questions, generates domain context."
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, TaskGet
user-invocable: true
---

# Create Context Skill

Create and maintain structured context files in `Context/` so Claude has rich domain knowledge in future sessions. Uses a hybrid approach: scans the vault for related content, then asks the user to confirm findings and fill gaps.

## Usage

```
/create-context              # Interactive — ask what context to create
/create-context new <domain> # Create new context file for named domain
/create-context update [name]# Update existing context file with new vault info
/create-context list         # List existing context files with metadata
```

Or naturally: "Create a context file for my company" / "Update my project context"

## Context File Template

Files are written to `Context/<Domain Name>.md`. Not all sections are required — only include sections with actual content.

```markdown
---
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [context]
type: context
domain: <domain-name>
---

# Context: [Domain Name]

## Overview
[What this domain covers and why it matters]

## Key Facts
[Essential, stable information — marked [solid]]

## Current State
[What's happening now — marked [evolving]]

## People & Relationships
[Key people, teams, contacts relevant to this domain]

## Active Work
[Current projects, initiatives, tasks in this domain]

## Decisions & Rationale
[Important decisions made and why]

## Preferences & Conventions
[How things are done, tools used, standards followed]

## Open Questions
[Unresolved items — marked [hypothesis] or [questioning]]

## Resources & Links
[Wiki-links to vault notes, external URLs]
```

### Confidence Markers

Apply these throughout each section:

| Marker | Meaning | When to use |
|--------|---------|-------------|
| `[solid]` | Confirmed fact | User confirmed or appears in multiple vault sources |
| `[evolving]` | Current but changing | Status info, current-state items that change frequently |
| `[hypothesis]` | Speculative | User flagged as uncertain or single-source inference |
| `[questioning]` | Being reconsidered | User is rethinking this or it may change soon |

---

## Command: `/create-context` or `/create-context new [domain]`

Creates a new context file. Runs 4 sequential steps with session task tracking.

### Session Tasks (create at start)

```
TaskCreate: subject="Determine domain & scan vault",   activeForm="Scanning vault for context..."
TaskCreate: subject="Present findings & ask questions", activeForm="Gathering domain details..."
TaskCreate: subject="Generate context file",            activeForm="Writing context file..."
TaskCreate: subject="Confirm & suggest next steps",     activeForm="Finalizing context..."
```

Set dependencies so each step blocks the next.

---

### Step 1: Determine Domain & Scan Vault

**Goal:** Identify what context to build and gather raw material from the vault.

#### 1a. Determine Domain

If a `<domain>` argument was provided, use it. Otherwise, ask the user:

```
AskUserQuestion:
  question: "What domain should this context cover?"
  options:
    - label: "Company / Employer"
      description: "Your workplace — role, team, objectives, org structure"
    - label: "Project"
      description: "A specific project — scope, goals, tech stack, decisions"
    - label: "Personal Workflow"
      description: "Your tools, routines, preferences, communication style"
    - label: "Technology"
      description: "A tech stack or tool — experience, patterns, learning goals"
```

The user can also pick "Other" for any custom domain.

#### 1b. Check for Existing Context

```
Glob: pattern="Context/<domain>*.md"
```

If a context file already exists for this domain:
- Tell user: "A context file already exists for `<domain>`. Would you like to update it instead?"
- If yes, switch to the **update** workflow
- If no, proceed (user may want a separate file)

#### 1c. Scan Vault

Search for domain-related content across the vault:

1. **Keyword search:** `Grep` for domain keywords (and common synonyms/abbreviations) across `*.md` files. Exclude `Admin/Templates/` and `Archives/`.
2. **Project context:** Check `Projects/*/CLAUDE.md` for mentions of the domain.
3. **Goals:** Check `Goals/*.md` files for references to the domain.
4. **Recent daily notes:** Read daily notes from the past 14 days, search for domain mentions.
5. **General notes:** Check `General Notes/*.md` for related content.
6. **Existing context files:** Check `Context/*.md` for cross-references.

Collect all findings into a grouped summary (by source type).

Mark Step 1 complete.

---

### Step 2: Present Findings & Ask Guided Questions

**Goal:** Show what was found and fill gaps interactively.

#### 2a. Present Vault Findings

Display what was found, grouped by source:

```markdown
### Vault Scan Results for "<domain>"

**Projects:**
- [[Projects/ProjectA/CLAUDE.md]] — mentions <domain> in 3 sections
- [[Projects/ProjectB/CLAUDE.md]] — references <domain> tools

**Goals:**
- [[Goals/1. Yearly Goals]] — "<domain>-related goal text"

**Daily Notes (past 14 days):**
- [[2026-02-25]] — discussed <domain> topic
- [[2026-02-22]] — captured notes about <domain>

**General Notes:**
- [[General Notes/some-note]] — detailed <domain> reference

**No matches in:** (list any categories with no results)
```

If nothing was found: "No vault content found for this domain. We'll build context from scratch."

#### 2b. Ask Domain-Specific Questions

Based on the domain type, ask targeted questions via `AskUserQuestion`. Pre-fill from vault findings when possible — user confirms or edits.

**Company/Employer:**
1. What is your role and title?
2. What team or department are you in?
3. What are your main objectives or responsibilities?
4. What tools and systems does the company use?
5. Who are the key people you work with?
6. What's the org structure around you?

**Project:**
1. What is the project's purpose and scope?
2. What phase is it in? (planning, active, maintenance)
3. What's the tech stack or toolset?
4. What key decisions have been made and why?
5. What are the current blockers or open questions?
6. Who else is involved?

**Personal Workflow:**
1. What does your typical day/week look like?
2. What tools do you use daily?
3. What's your communication style and preferences?
4. What are your productivity patterns? (best focus times, energy patterns)
5. What routines or rituals do you follow?

**Technology:**
1. What's your experience level with this technology?
2. What are your primary use cases?
3. What patterns or conventions do you follow?
4. What are you currently learning or exploring?
5. What resources do you reference?

**Custom domain:**
1. What is this domain and why does it matter to you?
2. What should Claude always know about this area?
3. What's the current state of things?
4. What are the key people, tools, or resources involved?
5. What's uncertain or evolving?

Ask 2-4 questions at a time using `AskUserQuestion` (not all at once). Adjust follow-up questions based on answers.

Mark Step 2 complete.

---

### Step 3: Generate Context File

**Goal:** Combine vault findings and user answers into a structured context file.

#### 3a. Build Content

For each section in the template:
- Combine relevant vault findings with user answers
- Apply confidence markers:
  - `[solid]` — facts confirmed by user OR appearing in 2+ vault sources
  - `[evolving]` — current-state info the user indicated changes frequently
  - `[hypothesis]` — items the user flagged as uncertain or speculative
  - `[questioning]` — things being actively reconsidered
- Add wiki-links (`[[Note Name]]`) to referenced vault notes
- Only include sections that have actual content — skip empty sections

#### 3b. Create Directory & Write File

```
Bash: mkdir -p "Context/"
Write: file_path="Context/<Domain Name>.md"
```

Use today's date for both `created` and `updated` frontmatter fields.

#### 3c. Minimal Fallback

If the user skipped all questions and no vault content was found, generate a minimal file with just:
- Frontmatter
- `## Overview` section with whatever the user did provide
- A note: `<!-- Expand this context by running /create-context update <domain> -->`

Mark Step 3 complete.

---

### Step 4: Confirm & Suggest Next Steps

**Goal:** Verify the output and guide next actions.

#### 4a. Display Summary

```markdown
### Context Created: <Domain Name>

- **File:** `Context/<Domain Name>.md`
- **Sections filled:** 6/9
- **Vault notes linked:** 4
- **Confidence:** 3 [solid], 2 [evolving], 1 [hypothesis]
```

#### 4b. Suggest Related Context

Based on what the user mentioned during Q&A, suggest additional context files:

- "You mentioned ProjectX — want to create a context file for it?"
- "Your company context references Team Y — a team context file could help."

#### 4c. Remind About Updates

"Run `/create-context update <domain>` periodically to keep this current. Daily notes and project updates will be incorporated automatically."

Mark Step 4 complete.

---

## Command: `/create-context update [name]`

Updates an existing context file with new vault information. Runs 3 sequential steps.

### Session Tasks (create at start)

```
TaskCreate: subject="Load existing context & scan",    activeForm="Scanning for changes..."
TaskCreate: subject="Present changes & confirm",       activeForm="Reviewing new information..."
TaskCreate: subject="Merge updates & write",           activeForm="Updating context file..."
```

Set dependencies so each step blocks the next.

---

### Step 1: Load Existing & Scan for Changes

**Goal:** Read current context and find what's new.

#### 1a. Select Context File

If no `[name]` argument provided:
1. `Glob` for `Context/*.md`
2. If no files exist, tell user: "No context files found. Run `/create-context new` to create one."
3. If files exist, list them and ask which to update

#### 1b. Read Existing Context

Read the context file. Parse:
- `updated` date from frontmatter
- `domain` from frontmatter
- All current content sections

#### 1c. Scan for New Information

Search for content added or changed since the `updated` date:

1. **Daily notes:** Read daily notes from `updated` date to today, search for domain mentions
2. **Project changes:** Check if `Projects/*/CLAUDE.md` files have been modified (compare content with what's referenced in the context file)
3. **Goal updates:** Check `Goals/*.md` for new references to the domain
4. **General notes:** Check `General Notes/*.md` for new domain-related content
5. **Other context files:** Check if other context files now reference this domain

Mark Step 1 complete.

---

### Step 2: Present Changes & Confirm

**Goal:** Show what's new and let the user decide what to include.

#### 2a. Display New Findings

```markdown
### Changes Since <updated date> for "<domain>"

**New daily note mentions:**
- [[2026-02-25]]: "Started using new deployment pipeline"
- [[2026-02-26]]: "Team standup moved to async"

**Project updates:**
- [[Projects/ProjectA/CLAUDE.md]]: Phase changed from Planning to Active

**No changes in:** Goals, General Notes
```

If no changes found: "Context is up to date. No new vault content found since <date>."
- Offer: "Want to add any manual updates?"
- If no, exit gracefully

#### 2b. Confirm Each Finding

For each finding, ask: keep / modify / skip

Use `AskUserQuestion` with grouped items when possible (don't ask one at a time for many items).

#### 2c. Manual Additions

Ask: "Any manual updates to add? (new facts, changed status, corrections)"

Mark Step 2 complete.

---

### Step 3: Merge & Write

**Goal:** Integrate confirmed changes into the context file.

#### 3a. Merge Content

For each confirmed change:
- Add to the appropriate section in the context file
- If a section doesn't exist yet, create it
- If content contradicts existing content, update it (don't duplicate)
- Update confidence markers if warranted:
  - `[hypothesis]` -> `[solid]` if now confirmed by multiple sources
  - `[evolving]` stays `[evolving]` unless user says it's settled
  - New items default to `[evolving]` unless clearly established

#### 3b. Update Frontmatter

Update `updated:` date to today.

#### 3c. Write File

```
Edit: file_path="Context/<Domain Name>.md"
```

Preserve existing content structure. Add new content in the appropriate sections.

#### 3d. Display Summary

```markdown
### Context Updated: <Domain Name>

- **Changes merged:** 4
- **Sections modified:** Current State, Active Work
- **New wiki-links added:** 2
- **Confidence upgrades:** 1 ([hypothesis] -> [solid])
- **Last updated:** 2026-02-27
```

Mark Step 3 complete.

---

## Command: `/create-context list`

Lists existing context files. Simple read-only operation — no session tasks needed.

### Steps

1. `Glob` for `Context/*.md`
2. If no files found: "No context files yet. Run `/create-context new` to create your first one."
3. For each file, read frontmatter: `domain`, `created`, `updated`
4. Calculate staleness:
   - Fresh (updated within 7 days)
   - Current (updated within 30 days)
   - Stale (updated 30+ days ago)
5. Display table:

```markdown
## Context Files

| Domain | Created | Last Updated | Status |
|--------|---------|--------------|--------|
| Company | 2026-02-15 | 2026-02-25 | Current |
| ProjectX | 2026-02-20 | 2026-02-20 | Stale |

Run `/create-context update <name>` to refresh a stale context file.
```

---

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| `Context/` doesn't exist | Create it automatically on first context file |
| Context file already exists for domain | Suggest `update` instead of `new` |
| No vault content found for domain | Proceed with pure Q&A (no pre-fill) |
| User skips all questions | Generate minimal file with just Overview |
| Update finds no changes | Tell user "Context is current" and offer manual additions |
| No context files exist (for list/update) | Friendly message suggesting `new` |

## Conventions

This skill follows all CLAUDE.md conventions:

- **Wiki-links:** Use `[[Note Name]]` format for all vault references
- **Frontmatter:** Include YAML frontmatter on all created files
- **Confidence markers:** `[solid]`, `[evolving]`, `[hypothesis]`, `[questioning]`
- **Heading structure:** H1 for title, H2 for sections, H3 for subsections
- **Tags:** Use `tags: [context]` in frontmatter

## Integration

- **Loads into:** The context-loading skill reads these files for deep domain knowledge
- **Informed by:** `/daily`, `/close-day`, `/weekly` — daily notes and reviews provide raw material
- **Complements:** `/onboard` — onboard loads CLAUDE.md context; create-context builds deeper domain files
- **Updates from:** `/project` — project status changes feed context updates
- **Cross-references:** Context files can wiki-link to each other for connected domains
