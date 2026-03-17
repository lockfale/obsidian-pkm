# Obsidian PKM Vault Context

## System Purpose
[CUSTOMIZE: Add your personal mission statement here]

*Example: "Build meaningful technology while maintaining balance across health, relationships, and personal growth."*

## Directory Structure

| Folder | Purpose |
|--------|---------|
| `Daily Notes/` | Daily journal entries (`YYYY-MM-DD.md`) |
| `Goals/` | Goal cascade (3-year → yearly → monthly → weekly) |
| `Projects/` | Active projects with their own `CLAUDE.md` |
| `Admin/Templates/` | Reusable note structures (Daily, Weekly, Project templates) |
| `Archives/` | Completed/inactive content |
| `General Notes/` | Uncategorized captures (optional) |

## Current Focus

See @Goals/2. Monthly Goals.md for this month's priorities.

## Tag System

**Priority:** `#priority/high`, `#priority/medium`, `#priority/low`
**Status:** `#active`, `#waiting`, `#completed`, `#archived`
**Context:** `#work`, `#personal`, `#health`, `#learning`, `#family`

## Available Skills

Skills are invoked with `/skill-name` or automatically by Claude when relevant.

| Skill | Invocation | Purpose |
|-------|------------|---------|
| `daily` | `/daily` | Create daily notes, morning/midday routines |
| `close-day` | `/close-day` | End-of-day processing — extract, reflect, roll tasks forward |
| `weekly` | `/weekly` | Run weekly review, reflect and plan |
| `monthly` | `/monthly` | Monthly review, quarterly milestone check, next month planning |
| `project` | `/project` | Create, track, and archive projects linked to goals |
| `review` | `/review` | Smart router — auto-detects daily/weekly/monthly based on context |
| `onboard` | `/onboard` | Interactive setup (first run) + load vault context |
| `upgrade` | `/upgrade` | Update to latest version, preserving your content |
| `goal-tracking` | (auto) | Track progress across goal cascade with project awareness |
| `create-context` | `/create-context` | Create and update structured domain context files for future sessions |
| `metrics` | `/metrics` | Measure ONE Big Thing hit rate and productivity metrics |
| `drift` | `/drift` | Compare intentions vs behavior, surface avoidance and push patterns |
| `obsidian-vault-ops` | (auto) | Read/write vault files, manage wiki-links |
| `adopt` | `/adopt` | Scaffold PKM system onto an existing Obsidian vault |
| `backlinks` | `/backlinks` | Analyze vault link structure, find orphans, score missing connections |
| `challenge` | `/challenge` | Push back on current thinking using vault history |
| `check-links` | `/check-links` | Find broken wiki-links in the vault (read-only) |
| `contradict` | `/contradict` | Find incompatible beliefs held simultaneously in the vault |
| `frontend-slides` | `/frontend-slides` | Create animation-rich HTML presentations from scratch or PPT |
| `ghost` | `/ghost` | Answer a question as the vault author would, evaluate fidelity |
| `ideas` | `/ideas` | Generate grounded, non-obvious ideas from vault patterns and gaps |
| `leverage` | `/leverage` | Find skills/knowledge where investment yields disproportionate breakthroughs |
| `push` | `/push` | Commit changes to git |
| `search` | `/search` | Search vault content by keyword using Grep |
| `trace` | `/trace` | Trace how an idea or belief evolved across the vault over time |

### Progress Visibility

Skills and agents use session task tools to show progress during multi-step operations:

```
[Spinner] Creating daily note...
[Spinner] Pulling incomplete tasks...
[Done] Morning routine complete (4/4 tasks)
```

Session tasks are temporary progress indicators—your actual to-do items remain as markdown checkboxes in daily notes.

## Available Agents

| Agent | Purpose |
|-------|---------|
| `note-organizer` | Organize vault, fix links, consolidate notes |
| `weekly-reviewer` | Facilitate weekly review aligned with goals |
| `goal-aligner` | Check daily/weekly alignment with long-term goals |
| `inbox-processor` | GTD-style inbox processing |

## Output Styles

**Productivity Coach** (`/output-style coach`)
- Challenges assumptions constructively
- Holds you accountable to commitments
- Asks powerful questions for clarity
- Connects daily work to mission

## The Cascade

The full goals-to-tasks flow:

```
3-Year Vision  →  Yearly Goals  →  Projects  →  Monthly Goals  →  Weekly Review  →  Daily Tasks
   /goal-tracking     /project        /project      /monthly          /weekly         /daily
```

## Daily Workflow

### Morning (5 min)
1. Run `/daily` to create today's note
2. Review cascade context (ONE Big Thing, project next-actions)
3. Identify ONE main focus
4. Review yesterday's incomplete tasks
5. Set time blocks

### Evening (5 min)
1. Run `/close-day` for guided end-of-day processing
2. Review vault connections and file captured content
3. Complete reflection section (interactive or drafted)
4. Set tomorrow's priorities and roll tasks forward

### Weekly (30 min - Sunday)
1. Run `/weekly` for guided review
2. Review project progress table
3. Calculate goal progress
4. Plan next week's focus
5. Archive old notes

### Monthly (30 min - End of month)
1. Run `/monthly` for guided review
2. Roll up weekly wins/challenges
3. Check quarterly milestones
4. Plan next month's focus

## Conventions

### Tomorrow's Top Priorities
When a task is identified as a top priority or ONE thing for tomorrow in the "Tomorrow's Top Priorities" section of a daily note:
- Find the original task in its source note
- Update its priority to highest (`🔺`)
- Update the `📅` date to tomorrow if not already set

Do not copy the task — update it in place, same as rolling tasks forward.

When creating or reviewing a daily note, check all tasks due that day with highest priority (`🔺`):
- If the task is listed in "Tomorrow's Top Priorities" or identified as the ONE thing → keep `🔺`
- If not → downgrade to high (`⏫`)

This keeps `🔺` meaningful — it should only ever reflect today's declared top priority, not carry over indefinitely.

### Weekly Review — Incomplete Tasks Section
The "Incomplete Tasks Carried Forward" section in a weekly review must use a Tasks query, never a manually copied list:

```tasks
not done
due before today
path does not include /Admin/Templates
path does not include Archives
sort by priority
```

Manually listing tasks creates duplicates that diverge from the source. Tasks live in their source notes; the query surfaces them dynamically.

### Rolling Tasks Forward
When carrying incomplete tasks to the next day:
- Update the `📅 YYYY-MM-DD` date on the original task **in its source note** to the new date
- Never copy tasks into today's Capture section
- Tasks with today's date surface automatically via the query blocks in the daily note
- This keeps task history intact and prevents duplicates

### Task Ownership
Every task checkbox lives in exactly ONE file — `Tasks.md`. Project tasks are organized under `## 🏗️ Project Tasks` by project, with each project having its own subsection. Project `CLAUDE.md` files use Obsidian Tasks queries (filtered by project tag) to surface their tasks dynamically. This keeps all tasks in a single source of truth.

**Project tags:** Add project tags here as they are created.

- **Daily note Capture = temporary inbox.** Tasks captured here get moved to their permanent home during `/close-day`. After filing, the task is deleted from the daily note.
- **To schedule a task for a specific day**, add `📅 YYYY-MM-DD` to the task in Tasks.md. Daily note queries auto-surface it.
- **To roll a task forward**, change the `📅` date in Tasks.md. Never copy.
- **All tasks must have a tag** (`#work`, `#personal`, `#learn`, etc.) — untagged tasks surface in the "Improperly Captured" section of Tasks.md.
- **Project tasks also need a project tag** (e.g., `#ProjectName`, `#SecondProjectName`) so they surface in project CLAUDE.md queries.
- **Feature backlog items** don't need dates — they're "someday" items, invisible to daily queries but visible via project tag queries. They still need a context tag and a project tag.
- **Only recurring Health & Habits items** live directly in Tasks.md health section (from the template).

### Monthly Review Archive
Before overwriting `Goals/2. Monthly Goals.md` during a monthly review, always archive the current file first:
- Destination: `Admin/Archives/Goals/YYYY-MM-Monthly Goals.md` (e.g., `Admin/Archives/Goals/2026-02-Monthly Goals.md`)
- Determine the month from the file's H1 heading
- Create `Admin/Archives/Goals/` if it doesn't exist
- Only skip archiving if the file contains only template placeholders (no real content)

### Weekly Review Archive
Before overwriting `Goals/3. Weekly Review.md` during a weekly review, always archive the current file first:
- Destination: `Admin/Archives/Goals/YYYY-WNN-Weekly Review.md` (e.g., `Admin/Archives/Goals/2026-W08-Weekly Review.md`)
- Determine the year and ISO week number from the file's H1 heading or "Week Number" field
- Create `Admin/Archives/Goals/` if it doesn't exist (shared with monthly archives)
- Only skip archiving if the file contains only template placeholders (no real content)

## Best Practices

1. **Be Specific** - Give clear context about what you need
2. **Reference Goals** - Connect daily tasks to objectives
3. **Use Coach Mode** - When you need accountability
4. **Keep It Current** - Update project CLAUDE.md files regularly

## Customization

For personal overrides that shouldn't be committed, create `CLAUDE.local.md`.
See `CLAUDE.local.md.template` for format.

---

*See @.claude/rules/ for detailed conventions*
*Last Updated: 2026-02-15*
*System Version: 3.1*


<claude-mem-context>
# Recent Activity

<!-- This section is auto-generated by claude-mem. Edit content outside the tags. -->

*No recent activity*
</claude-mem-context>