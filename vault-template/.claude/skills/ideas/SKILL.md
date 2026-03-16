---
name: ideas
description: "Generate grounded, non-obvious ideas by analyzing vault patterns, daily notes, goals, and structural gaps. Screen output only."
allowed-tools: Read, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet, AskUserQuestion
user-invocable: true
---

# Ideas Generation Skill

Generate a comprehensive list of ideas across multiple domains by analyzing vault structure, daily notes, context files, goals, and projects. Surfaces both obvious high-impact opportunities and non-obvious insights from pattern recognition. All output goes to screen only — no files created or modified.

## Usage

```
/ideas                    # Full 6-phase run
/ideas focus:work         # Constrain idea generation to work domain
/ideas focus:personal     # Constrain to personal domain
```

The `focus` argument constrains Step 4 category generation only — Steps 1-3 always run fully regardless of focus.

## Session Tasks (6 phases)

Create all 6 tasks upfront, then execute in order.

```
TaskCreate: subject="Explore vault relationships",   activeForm="Mapping vault structure and connections..."
TaskCreate: subject="Review deep context",            activeForm="Reading daily notes and context files..."
TaskCreate: subject="Analyze patterns",               activeForm="Synthesizing patterns across vault..."
TaskCreate: subject="Generate ideas",                 activeForm="Generating grounded ideas..."
TaskCreate: subject="Prioritize ideas",               activeForm="Ranking ideas by impact..."
TaskCreate: subject="Connect to exploration",         activeForm="Building exploration plan..."
```

### Dependencies

```
TaskUpdate: "Analyze patterns", addBlockedBy: [explore-vault-id, review-context-id]
TaskUpdate: "Generate ideas", addBlockedBy: [analyze-patterns-id]
TaskUpdate: "Prioritize ideas", addBlockedBy: [generate-ideas-id]
TaskUpdate: "Connect to exploration", addBlockedBy: [prioritize-ideas-id]
```

Task 3 (Analyze patterns) is blocked by both Task 1 and Task 2. All others are sequential.

---

## Step 1: Explore Vault Relationships

**Goal:** Structural analysis before reading individual files. Surfaces connections invisible from reading files in isolation.

### 1a. Collect All Files

```
Glob: pattern="**/*.md"
```

Exclude files under `.claude/`, `.obsidian/`, and `Admin/Templates/` from analysis.

### 1b. Extract Wiki-Links

```
Grep: pattern="\[\[([^\]|#^]+)" glob="**/*.md"
```

Build a map of all `[[link]]` targets across the vault.

### 1c. Find Orphans

Files that are never targeted by any `[[link]]` in the vault.

- Exclude: daily notes (`Daily Notes/*.md`), templates (`Admin/Templates/*.md`), CLAUDE.md files, vault config files
- Cap output at 15 orphans
- These represent forgotten ideas, neglected projects, or isolated thinking

### 1d. Find Deadends

Files with zero outgoing `[[links]]`.

- Exclude: daily notes older than 30 days
- Cap output at 10 deadends
- These contain valuable thinking that was never connected to anything

### 1e. Find Unresolved Links

`[[targets]]` referenced in the vault with no matching `.md` file.

- Group by reference count
- Show only targets with 2+ references
- These represent ideas worth creating notes for

### 1f. Tag Distribution

```
Grep: pattern="#[a-zA-Z][a-zA-Z0-9_/]+" glob="**/*.md"
```

Count and display top 15 tags. Shows where thinking is concentrated vs sparse.

### 1g. Backlink Chain Tracing

For the top 3-5 active themes identified so far:

- Trace connections 2-3 hops deep using targeted Grep searches
- Look for notes that share connections but aren't linked to each other (hidden relationships)
- Identify orphaned notes surprisingly relevant to current priorities

### 1h. Cross-Domain Pattern Detection

Flag themes appearing in 3+ vault directories. When separate areas of life point at the same thing, it's probably important.

### 1i. Display Structural Snapshot

Present a summary before proceeding:

```markdown
### Vault Structural Snapshot
- **Total files:** X
- **Orphans:** X (listed)
- **Deadends:** X (listed)
- **Unresolved links:** X (grouped by count)
- **Top tags:** [tag distribution]
- **Cross-domain themes:** [themes spanning 3+ directories]
- **Hidden relationships:** [notes sharing connections but not linked]
```

Mark Step 1 complete.

---

## Step 2: Deep Context Review

**Goal:** Extract signals from vault content — frustrations, energy sparks, recurring interests, unsolved problems, open questions.

### 2a. Daily Notes (Past 30 Days)

1. Glob all files in `Daily Notes/` matching `YYYY-MM-DD.md`
2. Filter to past 30 days from today's date
3. If fewer than 10 notes found, expand to 45 days, then 60 days
4. Read each note. **Skip `tasks` query blocks** (Obsidian-rendered, not useful as raw content)
5. Focus on these sections: Capture, Notes from Today, Reflection, Ideas & Thoughts

Extract:
- Frustrations and friction points
- Things that sparked energy or excitement
- Recurring interests or curiosities
- Problems mentioned but not solved
- Wishes ("I wish I could...", "It would be nice if...")
- Things tried and abandoned (and why)
- Questions asked but not answered

### 2b. Context Files

```
Glob: pattern="Context/**/*.md"
```

Read all context files. Extract:
- Open questions (explicitly listed)
- Items marked `[hypothesis]`, `[questioning]`, or `[evolving]`
- Constraints that might be worth challenging
- Goals without clear paths

### 2c. Goals & Projects

Read the goal cascade:
- `Goals/1. Yearly Goals.md`
- `Goals/2. Monthly Goals.md`
- `Goals/3. Weekly Review.md`

Read all project files:
```
Glob: pattern="Projects/*/CLAUDE.md"
```

Extract:
- Stalled progress or blocked goals
- Approaching milestones
- Feature backlogs
- "Not Doing" lists (explicitly deferred items)

### 2d. General Notes

```
Glob: pattern="General Notes/**/*.md"
```

Skim titles and first 20 lines of each file. Flag anything relevant to themes emerging from Steps 1-2.

### 2e. Temporal Tracking (Previous Runs)

Check if previous `/ideas` runs exist:

```
Grep: pattern="/ideas|ideas generation" path="Daily Notes"
```

If prior runs found:
- Which ideas got traction? (appeared in later daily notes, produced output)
- Which were abandoned? (never mentioned again)
- Which keep recurring? (persistent interests masquerading as ideas — need commitment, not another suggestion)

Use this to de-duplicate in Step 4.

### 2f. Display Extraction Summary

```markdown
### Context Extraction Summary
- **Daily notes scanned:** X (date range)
- **Context files read:** X
- **Projects reviewed:** X
- **Key frustrations:** [brief list]
- **Energy sparks:** [brief list]
- **Recurring interests:** [brief list]
- **Open questions:** [brief list]
- **Previous /ideas runs:** X found (Y ideas got traction, Z abandoned)
```

Mark Step 2 complete.

---

## Step 3: Pattern Analysis

**Goal:** Synthesize vault signals into actionable patterns before generating ideas.

Produce 5 categories with 3-7 bullets each. **Every bullet must cite its source** (`[[note name]]` or date).

### 1. What's Working

Things that consistently produce energy, output, or satisfaction.

### 2. What's Frustrating

Recurring friction, complaints, or energy drains.

### 3. What's Missing

Gaps between stated goals and current activities. Things desired but not pursued.

### 4. Recurring Interests

Topics, people, or tools appearing across 3+ notes.

### 5. Bottlenecks

Places where progress is repeatedly blocked, often by the same thing.

Mark Step 3 complete.

---

## Step 4: Generate Ideas

**Goal:** Produce specific, grounded, non-obvious ideas across 10 categories.

### Format

For each idea, include:
- **The idea** — specific and actionable, not generic
- **Vault evidence** — citation to `[[note]]` or date
- **Obvious vs Non-obvious** label
- **Effort estimate** — Quick (< 1 hour) / Medium (1 day–1 week) / Significant (> 1 week)
- **First concrete step** — what to do right now

### Categories

Generate 3-5 ideas per category:

1. **Tools to Build** — Custom tools, scripts, slash commands, or automations that solve specific identified problems
2. **Tools to Start Using** — Existing products, apps, or services addressing identified needs
3. **Systems to Implement** — Workflows, processes, or routines creating structure around recurring challenges
4. **Products to Purchase** — Physical items, subscriptions, or one-time purchases improving daily life
5. **Habits to Adopt** — Behavioral changes or practices to build
6. **Subjects to Investigate** — Topics worth going deep on, based on recurring interests or open questions
7. **Things to Write & Publish** — Essay ideas, threads, or content from unique perspectives in the vault
8. **Films to Watch** — Based on interests, themes in work, or creative fuel needs
9. **Conversations to Have** — People to reach out to, relationships to deepen
10. **Experiments to Run** — Low-cost tests of ideas, beliefs, or approaches

### Focus Constraint

If invoked with `focus:work` or `focus:personal`, constrain idea generation to that domain. Categories that don't apply to the focus domain should be skipped or filtered.

### Sparse Data Handling

If fewer than 3 ideas can be generated for a category, produce 1-2 and note the sparsity: "Limited vault data for this category."

### Temporal De-duplication

If an idea matches a previously abandoned suggestion (from Step 2e):
- Either skip it entirely
- Or reframe: "This was suggested on [date] and didn't get traction. What would need to change for it to happen this time?"

Mark Step 4 complete.

---

## Step 5: Prioritize

**Goal:** Surface the highest-impact ideas for action.

### Top 5 High-Impact, Do Now

The five ideas across all categories with the biggest positive impact that are actionable this week.

### Top 5 High-Impact, Requires Setup

Ideas that need more preparation but are worth investing in.

### Top 5 Non-Obvious Insights

Ideas that emerged from pattern recognition — not obvious from surface-level thinking.

### The One Thing

If you could only act on one idea from this entire list, which one would create the most positive change? Include reasoning that connects to vault evidence.

Mark Step 5 complete.

---

## Step 6: Connect to Exploration

**Goal:** Bridge ideas to the existing goal cascade and daily workflow.

### Tomorrow's Exploration

One specific action to take tomorrow. Be concrete: what to read, who to contact, what to build, what to try.

### This Week's Investigation

A subject or project to go deeper on this week. Connect to the weekly ONE Big Thing from `Goals/3. Weekly Review.md` if relevant.

### This Month's Experiment

A larger hypothesis to test over the coming weeks. Connect to monthly goals from `Goals/2. Monthly Goals.md` and active projects.

### Cascade Connection

Map the top recommendations to existing goal structures:
- Which weekly priorities do they align with?
- Which monthly goals do they advance?
- Which active projects do they support?
- Are any ideas worth becoming new projects?

Mark Step 6 complete.

---

## Output Guidelines

- **Be specific, not generic.** Every idea connects to something actually in the vault.
- **Cite sources.** "Based on your note from [date] about..." or "Referenced in [[note]]..."
- **Distinguish obvious vs non-obvious.** Both matter — label them.
- **Include effort estimates.** Quick / Medium / Significant.
- **Screen output only.** Do not create or modify any vault files. The user copies what they want.
- **This should feel like talking to someone who deeply understands your situation**, not reading a listicle.

## Integration

Works with:
- `/daily` — Ideas may inform tomorrow's priorities
- `/close-day` — Capture section may reference ideas to explore
- `/weekly` — Ideas feed into weekly planning and ONE Big Thing
- `/monthly` — Larger ideas map to monthly experiments
- `/project` — Some ideas may warrant new project creation
- `/create-context` — Context files are a primary input for pattern analysis
- Goal tracking skill — Ideas connect back to the goal cascade
