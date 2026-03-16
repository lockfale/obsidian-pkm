---
name: close-day
description: End-of-day processing — extract insights, file captured content, fill reflections, roll tasks forward. The evening counterpart to /daily.
allowed-tools: Read, Write, Edit, Glob, Grep, TaskCreate, TaskUpdate, TaskList, TaskGet
user-invocable: true
---

# Close Day Skill

End-of-day intelligence extraction and shutdown ritual. Reads today's daily note, discovers vault connections, files captured content, fills reflections interactively, and rolls incomplete tasks forward.

## Usage

```
/close-day          # Run full end-of-day processing
```

Or simply: "Close out the day" / "Evening shutdown" / "End of day"

## Date Resolution

Determine which day to close and where to roll tasks:

```
# Step 1: Determine TARGET_DATE
if (current hour < 4):
    TARGET_DATE = yesterday           # After midnight — close the day that just ended
elif (no daily note exists for today) AND (daily note exists for yesterday):
    TARGET_DATE = yesterday           # Next morning — close yesterday
else:
    TARGET_DATE = today               # Normal same-day close

# Step 2: Derive NEXT_DATE (always TARGET_DATE + 1)
NEXT_DATE = TARGET_DATE + 1 day
```

**NEXT_DATE is always relative to TARGET_DATE, never relative to the current date.**

- If closing yesterday → NEXT_DATE = today (not tomorrow)
- If closing today → NEXT_DATE = tomorrow

If TARGET_DATE ≠ today, confirm: "Closing out [TARGET_DATE]. Correct?"

## Step-by-Step Process

This skill runs 5 sequential steps with session task tracking. Create all 5 tasks upfront, then execute in order.

### Session Tasks (create at start)

```
TaskCreate: subject="Read & parse daily note",    activeForm="Reading today's daily note..."
TaskCreate: subject="Discover vault connections",  activeForm="Searching vault for connections..."
TaskCreate: subject="Extract & file content",      activeForm="Extracting and categorizing content..."
TaskCreate: subject="Fill reflections & metrics",   activeForm="Working through reflections..."
TaskCreate: subject="Carry forward & close",       activeForm="Rolling tasks forward..."
```

Set dependencies so each step blocks the next.

### Interaction Rule

**All interactive pauses in this skill MUST use plain text output, not AskUserQuestion.**
Present the question and options as formatted text, then **STOP your response completely**.
Wait for the user to reply in chat before continuing. Do not proceed, do not assume a default.
This applies to every step marked "interactive" below.

---

### Step 1: Read & Parse Daily Note

**Goal:** Understand the state of the day.

1. Read `Daily Notes/TARGET_DATE.md`
   - If it doesn't exist, stop and suggest running `/daily` first
2. Parse and summarize:
   - **Tasks done** — count `- [x]` items
   - **Tasks remaining** — count `- [ ]` items with `📅 TARGET_DATE`
   - **Tasks added** — count all task items (`- [ ]` and `- [x]`) in the Capture section that have actual content (not blank placeholders)
   - **Notes captured** — check if Notes from Today sections have content
   - **Reflection status** — check if End of Day Reflection sections are filled or empty
   - **Capture items** — check if Capture section has new items
3. Display a brief status summary:

```markdown
### Day Status: TARGET_DATE
- **Tasks:** 5/8 complete
- **Tasks added:** 3 new in Capture
- **Notes:** 3 sections with content (Meetings, General Notes)
- **Reflection:** Not yet filled
- **Captured items:** 2 new tasks in Capture section
```

Mark Step 1 complete.

---

### Step 2: Discover Vault Connections

**Goal:** Surface relevant connections, recurring themes, and shifts in thinking.

#### 2a. Extract Today's Themes

From the daily note content, identify 3-5 key themes/topics (people, projects, concepts, technologies mentioned).

#### 2b. Search for Connections

For each theme, run targeted searches:

```
Grep: pattern="<theme>" across vault (exclude Admin/Templates, Archives)
```

Also check:
- Which `Projects/*/CLAUDE.md` files mention these themes
- Which goal files reference these topics
- Which recent daily notes (past 14 days) discuss the same themes

#### 2c. Recurring Topic Detection

If a theme appears in 3+ daily notes within the past 14 days, flag it:

```markdown
**Recurring theme:** "API design" appeared in 4 notes this past 2 weeks
  → 02-14, 02-18, 02-22, 02-27
  Consider: Is this becoming a project? Does it need a dedicated note?
```

#### 2d. Confidence & Contradiction Check (simplified)

For themes that appear in both today's note and recent notes:
- Read the relevant recent entries
- Compare language/tone: Did today's writing shift from uncertain → definitive (or vice versa)?
- Check for direct contradictions: "On 02-20 you wrote X. Today you wrote Y."

Flag only clear, actionable shifts — not every minor wording difference.

#### 2e. Suggest Wiki-Links

Identify terms in today's note that match existing note names but aren't linked:
- People mentioned → existing people notes
- Projects mentioned → existing project folders
- Concepts → existing notes

#### 2f. Display Results

Show top 3-5 connections only (not exhaustive). Format:

```markdown
### Vault Connections
1. **"API design"** — connects to [[Projects/MyApp]] and [[2026-02-22]] (recurring: 4x in 2 weeks)
2. **"team scaling"** — relates to [[Goals/1. Yearly Goals#Build team capacity]]; tone shifted from questioning (02-18) to confident today
3. **"Redis caching"** — mentioned in [[Projects/Infrastructure]] but no link in today's note

**Suggested wiki-links to add:** [[Projects/MyApp]], [[John Smith]]
```

If no meaningful connections found, say so briefly and move on.

Mark Step 2 complete.

---

### Step 3: Extract & File Content

**Goal:** Categorize captured content and suggest where it should live.

#### 3a. Scan Capture Sections

Read these sections of the daily note:
- `### Capture — New Tasks` (new tasks added during the day)
- `### Ideas & Thoughts`
- `### General Notes`
- `### Meetings`

#### 3a'. Scan Notes for Uncaptured Tasks

**Goal:** Surface implicit tasks buried in meeting notes, general notes, and ideas that weren't formally captured as checkboxes.

Scan the content of these sections for action-oriented language:
- `### Meetings`
- `### General Notes`
- `### Ideas & Thoughts`

**Detection patterns** (case-insensitive):
- Explicit intent: "I need to", "I should", "I will", "I want to", "we need to", "we should"
- Commitment language: "follow up", "reach out to", "send", "schedule", "set up"
- Future work: "at some point", "eventually", "next step", "TODO", "to-do"
- Action verbs in context: "build", "create", "investigate", "test", "review", "update", "fix"

**What to flag:**
- Sentences or bullet points matching the patterns above
- Incomplete meeting notes (trailing dashes or truncated sentences) — flag as "Meeting notes may be incomplete"
- Detailed goals/plans listed as prose or sub-bullets that lack corresponding tasks

**What to skip:**
- Past-tense statements ("I completed", "we finished") — these are records, not tasks
- Items already captured as `- [ ]` tasks in the Capture section
- Pure observations or references with no action implication

**Present as a separate table before the Capture task triage:**

```markdown
### Potential Uncaptured Tasks (from Notes)

| Source | Text | Suggested Task |
|--------|------|----------------|
| Meetings (Carlos 1:1) | "Better end of investigation tracking for trends" | `- [ ] Define PRISM investigation tracking requirements #work` |
| General Notes | "I should think through how to share templates" | `- [ ] Plan ObsidianPKM sharing workflow (templates, skills, demo vault) #work #AI` |
| Meetings (Jason 1:1) | *(incomplete — trailing dash)* | ⚠️ Meeting notes incomplete — review and capture any actions |

Want to add any of these to today's Capture for filing?

Options:
- **Add all** — add all suggested tasks to Capture
- **Add selected** — tell me which ones (by number)
- **Skip** — no uncaptured tasks
```

**This step is interactive.** Present the table above as text output, then STOP and wait for the user to reply. Do not use AskUserQuestion. Do not continue until the user responds.

**Important:** This step is advisory. It surfaces candidates — the user decides what becomes a task. Do not auto-create tasks from prose.

#### 3b. Categorize Items

Sort each captured item into:

| Category | Examples |
|----------|----------|
| **Action** | Tasks, follow-ups, commitments made |
| **Idea** | Original thoughts, experiment ideas, content ideas |
| **Commitment** | Promises to others, deadlines mentioned |
| **Question** | Open questions, things to investigate |
| **Reference** | Facts, links, resources to save |

#### 3b'. Task Triage (Capture section tasks)

**Completed tasks (`- [x]`) in Capture: leave in place.** These are records of ad-hoc work done that day. They don't need filing — the `### ✅ Completed Today` query surfaces them automatically, and they serve as history for weekly reviews and reflection.

For each **open** `- [ ]` item in the Capture section:

1. **Detect project affiliation** — match task text and tags against active `Projects/*/CLAUDE.md` files
2. **Check for duplicates** — fuzzy-match task text against existing tasks in the detected project file. If a near-identical task exists, WARN instead of filing.
3. **Present triage table:**

| Task | Destination | Duplicate? |
|------|-------------|------------|
| "SIA Metrics into scoring" | Tasks.md → Project Tasks → Account Scoring (#AS) | No |
| "Laundry" | Tasks.md → Non-Project Tasks | No |

4. **Confirm before filing** — this step is interactive:

   Present the triage table above as text output, then display:

   "Here's where I'd file these tasks. Proceed with all, adjust any, or skip?"
   - **Proceed** — file all as shown
   - **Adjust** — tell me which to change (by number)
   - **Skip** — leave tasks in Capture

   Then STOP and wait for the user to reply. Do not use AskUserQuestion. Do not continue until the user responds.

   **Do not write to any vault file until the user confirms.**

5. **After confirmation, execute filing:**
   - Project tasks → append to `Tasks.md` `## 🏗️ Project Tasks` → appropriate project subsection
   - Non-project tasks → append to `Tasks.md` `## ✏️ Non-Project Tasks` section
   - Preserve all metadata (tags, dates, priority)
   - Delete the **open** task from the daily note Capture section (completed tasks stay)

#### 3c. Suggest Filing Locations

Present non-task items as a table:

```markdown
### Filing Suggestions

| Item | Category | Suggested Location | Action |
|------|----------|--------------------|--------|
| "Try Redis for session cache" | Idea | [[Projects/Infrastructure]] | Add to ideas section |
| "Follow up with Sarah on budget" | Commitment | Tasks.md | Create task with due date |
| "Team standup moved to 10am" | Reference | [[Projects/TeamOps]] | Update meeting schedule |
```

**Action** items affiliated with a project go to `Tasks.md` → `## 🏗️ Project Tasks` → the project's subsection.

#### 3d. Confirm Before Filing

**This step is interactive.** Present the table above as text output, then display:

"Here's where I'd file these items. Proceed, adjust, or skip?"
- **Proceed** — file all as shown
- **Adjust** — tell me which to change (by number)
- **Skip** — leave items in the daily note

Then STOP and wait for the user to reply. Do not use AskUserQuestion. Do not continue until the user responds.

Only write to vault files after confirmation.

When filing:
- For actions/commitments: Create properly formatted tasks with tags and `📅` dates
- For ideas: Append to the relevant note's appropriate section
- For references: Add to the relevant note with a date stamp

Mark Step 3 complete.

---

### Step 4: Reflection & Metrics

**Goal:** Fill empty reflection sections, capture energy levels, set tomorrow's priorities.

#### 4a. Check What's Already Filled

Read the `## 🔍 End of Day Reflection` section. For each subsection, check if it has content beyond the template placeholder.

#### 4b. Fill Empty Sections — Two-Phase Reflection

**Phase 1 — Prompt the user (one question at a time):**

For each empty reflection section, present the question as text output, then STOP and wait for the user to reply in chat. Do not use AskUserQuestion. Do not ask the next question until the current one is answered and written.

1. "**What went well today?** (or type 'skip')" → write answer to `### What Went Well?`
   STOP. Wait for reply. Write answer to daily note. Then ask next question.
2. "**What could be better?** (or type 'skip')" → write answer to `### What Could Be Better?`
   STOP. Wait for reply. Write answer to daily note. Then ask next question.
3. "**What did you learn?** (or type 'skip')" → write answer to `### What Did I Learn?`
   STOP. Wait for reply. Write answer to daily note.

**Phase 2 — Offer additional observations:**

After all user answers are written, analyze the day's content (completed tasks, incomplete tasks, themes, blockers, ideas captured) and generate additional observations the user may not have considered.

Present as text output: "**Additional observations from today's activity:**"
- Wins the user didn't mention (e.g., completed tasks, progress on goals)
- Challenges worth noting (e.g., recurring blockers, tasks rolled forward multiple times)
- Learning signals (e.g., new topics explored, meeting takeaways, ideas captured)

Then display:
"Want to append any of these to your reflections? **Add all**, **select by number**, or **skip**"

STOP and wait for the user to reply. Do not use AskUserQuestion. Do not continue until the user responds.

Only update the daily note again if the user confirms additions.

#### 4c. Energy Levels

If the energy section is empty (`/10`), present as text output:

"**Energy check** — give me three numbers (1-10) or type 'skip':
Focus / Physical / Emotional (e.g. `7 5 6`)"

STOP and wait for the user to reply. Do not use AskUserQuestion. Do not continue until the user responds.

Fill in the values from their reply.

#### 4d. Tomorrow's Priorities

If `### Tomorrow's Top Priorities` is empty:

1. Surface context:
   - Incomplete tasks from today
   - Upcoming deadlines (check tasks with `📅` dates in next 3 days)
   - Active project next-actions from `Projects/*/CLAUDE.md`
   - Weekly ONE Big Thing from `Goals/3. Weekly Review.md`

2. Present as text output: "**What's your ONE priority for tomorrow?** Here's what's on deck:"
   - List the top candidates with numbers
   - End with: "Pick a number, name your own, or type 'skip'"

   STOP and wait for the user to reply. Do not use AskUserQuestion. Do not continue until the user responds.

3. When confirmed:
   - Fill the Tomorrow's Top Priorities section
   - Find the original task in its source note
   - Update its priority to `🔺` (highest)
   - Update `📅` to tomorrow's date if not set
   - Per CLAUDE.md conventions: update in place, never copy

Mark Step 4 complete.

---

### Step 5: Carry Forward & Close

**Goal:** Roll incomplete tasks, generate impact summary, close out.

#### 5a. Identify Incomplete Tasks

Find all tasks with `📅 TARGET_DATE` that are still `- [ ]`:
- Search daily note for unchecked items
- Search vault-wide for tasks due TARGET_DATE: `Grep: pattern="📅 TARGET_DATE" glob="*.md"`

Exclude tasks in `Admin/Templates/` and `Archives/`.

#### 5b. Confirm Rollforward

**This step is interactive.** Present incomplete tasks as a numbered list with source info, then display:

```markdown
### Incomplete Tasks to Roll Forward

1. [ ] Draft API spec 📅 2026-02-27 — *source: Projects/MyApp/tasks.md*
2. [ ] Review chapter 3 📅 2026-02-27 — *source: Daily Notes/2026-02-25.md*
3. [ ] Fix login bug 📅 2026-02-27 — *source: Tasks.md*

Roll all to NEXT_DATE (2026-02-28)?
- **Roll all** — update all dates to NEXT_DATE
- **Select** — tell me which by number (drop the rest)
- **Drop all** — leave as overdue
```

**Important:** Always display the actual NEXT_DATE so the user can verify. If closing a previous day, NEXT_DATE should be today — not tomorrow.

Then STOP and wait for the user to reply. Do not use AskUserQuestion. Do not continue until the user responds.

#### 5c. Execute Rollforward

For confirmed tasks, update `📅` dates **in the source note** (not the daily note):
- Change `📅 TARGET_DATE` → `📅 NEXT_DATE`
- **NEXT_DATE = TARGET_DATE + 1** (see Date Resolution above). When closing a previous day, this means rolling to today, not tomorrow.
- Per CLAUDE.md conventions: never copy tasks, always update in place
- If a task was marked as tomorrow's priority in Step 4, ensure it has `🔺`

#### 5d. Fill Daily Metrics

Update the `## 📊 Daily Metrics` section in the daily note:

- **Tasks Completed:** Single number — count of `- [x]` tasks completed today (vault-wide, from the count already computed in Step 1)
- **Tasks Added:** Single number — count of task items (`- [ ]` and `- [x]`) in the `### ✏️ Capture — New Tasks` section that have actual content (not blank placeholders)

Write values directly — e.g., `- Tasks Completed: 5` and `- Tasks Added: 3`.

If the `## 📊 Daily Metrics` section doesn't exist in the note (older notes), skip silently.

#### 5e. Cascade Impact Summary

Generate a summary of what received attention today:

```markdown
### Today's Cascade Impact
- **Goals touched:** [[Goal 1]] (2 tasks), [[Goal 3]] (1 task)
- **Projects advanced:** [[ProjectA]] (3 tasks done), [[ProjectB]] (1 task)
- **Unlinked tasks:** 2 (consider linking to a goal or project)
- **Themes explored:** API design, team scaling
```

Build this by:
- Cross-referencing completed tasks with their project/goal links
- Counting tasks per project/goal
- Noting any tasks not linked to a goal or project

#### 5f. Closing Summary

Display a final compact summary:

```markdown
---
### Day Closed: TARGET_DATE

✓ 5/8 tasks completed
✓ 3 items filed to vault
✓ Reflections filled
✓ 3 tasks rolled to tomorrow
✓ Tomorrow's priority: "Draft API spec"

**Cascade:** 2 goals touched, 2 projects advanced
**Vault:** 2 new connections discovered, 1 recurring theme flagged

---
```

Mark Step 5 complete.

---

## Skipping Logic

The skill should gracefully handle missing content:

| Condition | Behavior |
|-----------|----------|
| No daily note exists | Stop after Step 1, suggest `/daily` |
| No notes captured | Skip Step 3 extraction (still check Capture section for tasks) |
| Reflection already filled | Skip Step 4 prompts for filled sections, only ask about empty ones |
| No incomplete tasks | Skip Step 5 rollforward, still show impact summary |
| No vault connections found | Brief "No strong connections today" message, move on |

## Conventions

This skill follows all CLAUDE.md conventions:

- **Task rollforward:** Update `📅` in source note, never copy tasks
- **Tomorrow's priority:** Set `🔺` on the original task, update in place
- **Priority decay:** Tasks carrying `🔺` from a previous day that aren't declared as tomorrow's priority get downgraded to `⏫`
- **Wiki-links:** Use `[[Note Name]]` format for suggestions
- **Filing:** Always confirm before writing to vault files outside the daily note

## Integration

Works with:
- `/daily` — Morning startup; `/close-day` is the evening counterpart
- `/weekly` — Weekly review uses daily reflections and cascade data
- `/review` — Routes to `/close-day` when invoked after 5 PM
- `/project` — Surfaces project next-actions for tomorrow's planning
- Goal tracking skill — Cascade impact summary connects to goal progress
