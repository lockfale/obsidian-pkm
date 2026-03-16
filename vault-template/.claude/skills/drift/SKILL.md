---
name: drift
description: "Compare stated intentions against actual behavior. Surface avoidance, broken commitments, and recurring push patterns. Screen output only."
allowed-tools: Read, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet, AskUserQuestion
user-invocable: true
---

# Drift -- Compare Intentions vs. Behavior

Compare stated intentions against actual behavior over a configurable time range. Surface gaps between what's declared as important and where time and energy actually go. Find what's being avoided, what keeps getting pushed, and where commitments have quietly broken down.

## Usage

```
/drift                    # Default -- past 60 days
/drift 30                 # Custom range -- past 30 days
/drift <topic>            # Focused -- drift analysis on one priority area
```

Examples:
- `/drift` -- full drift analysis, 60-day window
- `/drift 30` -- narrower window
- `/drift AI integration` -- focused on a specific priority
- `/drift career` -- focused on a goal area

If the argument is a number, treat it as the day range. Otherwise, treat it as a topic filter.

## Session Tasks (4 phases)

Create all 4 tasks upfront, then execute in order.

```
TaskCreate: subject="Gather stated intentions",   activeForm="Reading goals, projects, and context files..."
TaskCreate: subject="Gather actual behavior",      activeForm="Scanning daily notes and task patterns..."
TaskCreate: subject="Analyze drift",               activeForm="Scoring alignment and detecting avoidance..."
TaskCreate: subject="Generate drift report",       activeForm="Building drift report..."
```

### Dependencies

```
TaskUpdate: "Gather actual behavior", addBlockedBy: [intentions-id]
TaskUpdate: "Analyze drift", addBlockedBy: [behavior-id]
TaskUpdate: "Generate drift report", addBlockedBy: [analyze-id]
```

All tasks are sequential.

---

## Phase 1: Gather Stated Intentions

**Goal:** Build a complete picture of what was declared as important -- goals, commitments, priorities, and open questions.

### 1a: Read the Goal Cascade

```
Read: Goals/0. Three Year Goals.md
Read: Goals/1. Yearly Goals.md
Read: Goals/2. Monthly Goals.md
Read: Goals/3. Weekly Review.md
```

Extract:
- Stated priorities and goals at each level
- Commitments made (to self and others)
- Things marked as important or urgent
- ONE Big Thing declarations from weekly reviews

### 1b: Read Context Files

```
Glob: pattern="Context/**/*.md"
```

Read all context files. Extract:
- Open questions flagged for investigation
- Experiments proposed but not yet run
- Items marked `[evolving]` or `[hypothesis]`
- Stated focus areas and strategic bets

### 1c: Read Project Files

```
Glob: pattern="Projects/*/CLAUDE.md"
```

Read each project CLAUDE.md. Extract:
- Projects in active status
- Next actions listed
- Stated project goals and milestones
- Blockers or dependencies noted

### 1d: Scan Daily Notes for Stated Intentions

```
Glob: pattern="Daily Notes/YYYY-MM-*.md"   # across the configured range
```

Read daily notes within the time range. Look for:
- "I need to..." / "I should..." / "I want to..." statements
- Carry-forward items from reflections
- Things mentioned as tomorrow's priority
- Recurring items that keep appearing in capture sections
- ONE Big Thing entries

### Sparse Data Handling

| Condition | Behavior |
|-----------|----------|
| < 3 context files | Note reduced confidence; lean on goals and daily notes |
| < 7 daily notes in range | Expand range by 50%; note limited behavioral data |
| Topic argument has 0 mentions | Report "No mentions found" with synonym suggestions |

Mark Phase 1 complete.

---

## Phase 2: Gather Actual Behavior

**Goal:** Measure where time and energy actually went -- task patterns, daily note content, and vault activity.

### 2a: Task Completion Patterns

For each daily note in the range, scan for task markers:

- Count completed tasks: `- [x]`
- Count incomplete tasks with date changes (push detection): tasks where `📅` dates were updated to later dates
- Count tasks marked with highest priority `🔺` -- did they get completed that day?
- Track ONE Big Thing hit rate: days where the declared ONE Big Thing was completed vs. not

### 2b: Daily Note Content Analysis

Read each daily note. Analyze:

- **Time block patterns**: What categories of work appear in time blocks? How much deep work vs. meetings vs. admin?
- **Capture sections**: What topics appear in captures? What gets energy (long entries, detail, ideas flowing)?
- **Reflection sections**: What themes recur in reflections? What's celebrated vs. what's flagged as incomplete?
- **Topics that appear once and vanish**: Mentioned briefly, no follow-through

### 2c: Vault Activity Frequency

For each stated priority from Phase 1, measure actual engagement:

```
Grep: pattern="<stated priority A>" glob="Daily Notes/*.md" output_mode="files_with_matches"
Grep: pattern="<stated priority B>" glob="Daily Notes/*.md" output_mode="files_with_matches"
```

Count how many daily notes mention each stated priority. A priority mentioned in 2 of 60 daily notes has very different engagement than one mentioned in 40 of 60.

### 2d: Project Activity

For each active project, check:

```
Grep: pattern="<project name>" glob="Daily Notes/*.md" output_mode="files_with_matches"
```

- How many daily notes reference this project?
- Are next actions getting completed or accumulating?
- When was the project CLAUDE.md last meaningfully updated?

### 2e: Push Pattern Detection

Scan for tasks that have been rescheduled multiple times:

```
Grep: pattern="📅" glob="Projects/*/CLAUDE.md"
Grep: pattern="📅" glob="Tasks.md"
```

Read these files and identify tasks where:
- The task is still incomplete (`- [ ]`)
- The `📅` date is in the past or has been changed multiple times
- The same task text appears across multiple daily notes (carried forward repeatedly)

Mark Phase 2 complete.

---

## Phase 3: Analyze Drift

**Goal:** Compare Phase 1 (intentions) against Phase 2 (behavior) to score alignment, detect avoidance, and find recurring patterns.

### 3a: Alignment Scoring

For each stated priority from Phase 1, assign an alignment score (1-10):

- **9-10**: Priority gets consistent daily attention, tasks complete on schedule, vault shows active engagement
- **7-8**: Regular attention with some gaps, most tasks complete
- **5-6**: Sporadic attention, some tasks pushed, mentioned but not consistently acted on
- **3-4**: Rarely appears in daily notes, tasks accumulate, mostly lip service
- **1-2**: Effectively abandoned -- stated as priority but zero meaningful engagement

Score based on evidence from Phase 2: daily note mentions, task completion rates, time block allocation, and project activity.

### 3b: Unplanned Attention Sinks

Identify topics/activities that consumed significant time but do NOT appear in any stated priority, goal, or project:

- Frequent daily note topics not connected to goals
- Time blocks spent on unlisted activities
- Tasks completed that don't trace back to any project or goal

These aren't necessarily bad -- but they represent where energy actually went vs. where it was supposed to go.

### 3c: Avoidance Detection

For each item with alignment score <= 4, run the avoidance decision tree:

**(a) Mentioned in the past 7 days?**
- Yes = **active avoidance** (it's on your mind but not getting action)
- No = **passive avoidance** (quietly dropped from awareness)

**(b) Related items getting done?**
- If yes = the avoidance isn't about capacity. It's **specific avoidance** of THIS thing. Dig into why.
- If no = may be a broader capacity issue

**(c) Has a clear next step?**
- If no = avoidance may be about **ambiguity**, not resistance. The fix is defining one concrete action.
- If yes = the resistance is real, not structural

**(d) Requires someone else?**
- If yes = the block may be **relational**, not motivational. Identify who and what the ask is.
- If no = the block is internal

### 3d: Recurring Push Patterns

From Phase 2e push detection, for each identified push loop:

- How many times has this task been rescheduled?
- Over what time span?
- What typically happens on the days it's scheduled? (Other tasks completed? Nothing done? Different priorities take over?)
- What has previously broken similar loops? (External deadline, someone else asking, reframing the task)

### 3e: ONE Big Thing Analysis

Calculate the ONE Big Thing hit rate across the range:

- Days with a declared ONE Big Thing: [count]
- Days where it was completed: [count]
- Hit rate: [percentage]
- Pattern: Are misses clustered around certain topics or project areas?

Mark Phase 3 complete.

---

## Phase 4: Generate Drift Report

**Goal:** Render the structured drift report to screen.

### Output Format

```
DRIFT REPORT -- [start date] to [end date] ([N] days)
Daily notes analyzed: [count]
Active projects: [count]
Stated priorities: [count]

---

## Alignment Table

| Priority | Stated Importance | Actual Time/Energy | Alignment Score | Drift Level |
|----------|------------------|--------------------|-----------------|-------------|
| ...      | High             | Low                | 3/10            | Significant |

## ONE Big Thing Hit Rate

[X]% ([completed]/[declared] days)
Pattern: [observations about when misses cluster]

## What's Getting Unplanned Attention

[Topics/activities consuming time that aren't in any priority list. For each:
what it is, how much time it got, whether it should be formalized as a priority
or recognized as a distraction]

## What's Being Avoided

For each avoided item:

### [Item Name]
- **Evidence:** [How many times mentioned vs. how much action taken, with specific dates]
- **Avoidance type:** [Active/Passive] avoidance
- **Decision tree:** (a) [recent mention?] (b) [related items done?] (c) [clear next step?] (d) [requires someone else?]
- **Diagnosis:** [Ambiguity / Resistance / Relational block / Capacity]
- **Cost of continued avoidance:** [What's this costing you?]

## Recurring Push Patterns

| Item | Times Pushed | Time Span | What Breaks the Loop |
|------|-------------|-----------|---------------------|
| ...  | 4           | 3 weeks   | External deadline   |

## The Honest Assessment

[A direct, compassionate summary of where the drift is. Not judgmental. Just clear.
"Here's what you said mattered. Here's what you actually did. Here's where they
don't match." Cite specific dates and notes as evidence. Distinguish between
busyness and avoidance. Acknowledge legitimate reprioritization vs. quiet abandonment.]

## Recommended Corrections

For each significant drift item, one of:

- **Drop it**: If it keeps getting avoided, maybe it's not actually a priority. Remove it honestly.
- **Do it now**: If it genuinely matters, identify the specific next action and when.
- **Delegate it**: If it matters but you won't do it, who else can?
- **Reframe it**: If the next step is unclear, define one concrete action that would break the stall.
```

Mark Phase 4 complete.

---

## Output Guidelines

- **Be honest but not harsh.** This is a mirror, not a critic.
- **Always cite specific dates, notes, and data.** Vague drift reports are useless.
- **The most valuable insight is often the simplest:** "You said X matters. You haven't touched it in 3 weeks."
- **Don't confuse busyness with avoidance.** Some things legitimately got deprioritized. The real drift is things that STILL matter but aren't getting attention.
- **Distinguish between "dropped and should stay dropped" vs. "dropped and it's costing you."**
- **Screen output only.** Do not create or modify any vault files. The user copies what they want.

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| < 7 daily notes in range | Report "Insufficient daily note data for drift analysis" with suggestion to expand range |
| No goals files found | Report "No goal cascade found -- drift analysis requires stated intentions to compare against" |
| Topic argument has 0 mentions | Report "No mentions found for [topic]" with synonym suggestions |
| All priorities score 7+ alignment | Report honestly -- note that alignment is strong, highlight any minor gaps or unplanned sinks |

## Integration

Works with:
- `/challenge` -- Challenge results may reveal beliefs driving avoidance patterns
- `/trace` -- Trace the evolution of a repeatedly-pushed item
- `/metrics` -- Metrics provides the quantitative data; drift provides the qualitative interpretation
- `/review` -- Drift findings inform weekly and monthly reflection
- `/weekly` -- Weekly review is a natural time to run drift analysis
