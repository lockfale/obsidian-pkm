---
name: metrics
description: "Measure PKM system effectiveness. Tracks ONE Big Thing hit rate and other productivity metrics. Screen output only."
allowed-tools: Read, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet
user-invocable: true
---

# Metrics — ONE Big Thing Hit Rate

Measure whether daily planning translates to daily execution. Scans the vault for all `🔺` tasks, compares scheduled dates to completion dates, and reports hit rate.

## Usage

```
/metrics                    # Full report (all time)
/metrics period:week        # Last 7 days only
/metrics period:month       # Last 30 days only
```

## Session Tasks (3 phases)

Create all 3 tasks upfront, then execute in order.

```
TaskCreate: subject="Collect tasks",   activeForm="Scanning vault for top-priority tasks..."
TaskCreate: subject="Analyze results", activeForm="Calculating hit rate and patterns..."
TaskCreate: subject="Report metrics",  activeForm="Generating metrics report..."
```

### Dependencies

```
TaskUpdate: "Analyze results", addBlockedBy: [collect-id]
TaskUpdate: "Report metrics", addBlockedBy: [analyze-id]
```

---

## Step 1: Collect Tasks

**Goal:** Find every `🔺` task in the vault with a `📅` date.

### 1a: Grep for Tasks

Search for lines containing both `🔺` and `📅`:

```
Grep: pattern="🔺.*📅" glob="*.md" output_mode="content"
```

### 1b: Filter Results

From the grep results, keep only lines that are actual task checkboxes:
- Must start with `- [x]` or `- [ ]`
- Exclude files in `Admin/Templates/` (template placeholders)
- Exclude files in `.claude/` (skill definitions, rules)
- Exclude lines that are metric definitions (e.g., lines describing what `🔺` means rather than actual tasks)

A valid task line looks like:
```
- [x] Description #tags 🔺 📅 YYYY-MM-DD ✅ YYYY-MM-DD
- [ ] Description #tags 🔺 📅 YYYY-MM-DD
```

### 1c: Period Filter

If the user specified a period:
- `period:week` — only include tasks where `📅` date is within the last 7 days
- `period:month` — only include tasks where `📅` date is within the last 30 days
- No period specified — include all tasks

Mark Step 1 complete.

---

## Step 2: Analyze

**Goal:** Parse each task and classify as hit, late hit, or miss.

### 2a: Extract Fields

For each task line, extract:
- **Status:** `[x]` (done) or `[ ]` (not done)
- **Description:** The task text between the checkbox and the first emoji/tag
- **Tags:** All `#tag` values
- **Scheduled date:** `📅 YYYY-MM-DD`
- **Completion date:** `✅ YYYY-MM-DD` (if present)

### 2b: Classify

| Classification | Condition |
|---------------|-----------|
| **Hit** | `✅` date == `📅` date (completed same day as scheduled) |
| **Late hit** | `✅` date exists AND `✅` date > `📅` date (completed, but after scheduled day) |
| **Miss** | No `✅` date, or status is `[ ]` (not completed) |

Note: If `✅` date < `📅` date (completed before scheduled), classify as **Hit** — early completion is a win.

### 2c: Calculate Metrics

- **Hit rate:** hits / total (the primary metric)
- **Late rate:** late hits / total
- **Miss rate:** misses / total
- **Completion rate:** (hits + late hits) / total
- **Average delay:** For late hits, average number of days between `📅` and `✅`

### 2d: Group by Week

For each ISO week that contains at least one `🔺` task:
- Count tasks, hits, and calculate rate
- Use the `📅` date to assign week

### 2e: Group by Tag

For each unique tag found on `🔺` tasks:
- Count tasks and calculate hit rate
- Common tags: `#work`, `#personal`, `#learning`

Mark Step 2 complete.

---

## Step 3: Report

**Goal:** Output a formatted report to screen. Do NOT create or modify any vault files.

### Output Format

```
METRICS: ONE Big Thing Hit Rate
Period: YYYY-MM-DD to YYYY-MM-DD
Total 🔺 tasks: N

Hit Rate:  X% (N/N same-day completions)
Late Rate: X% (N/N completed but not same day)
Miss Rate: X% (N/N not completed)
Target:    70%+

---

## Weekly Trend
| Week | Tasks | Hits | Rate |
|------|-------|------|------|
| W08  |   2   |  2   | 100% |
| W09  |   3   |  2   |  67% |
...

## By Domain
| Tag       | Tasks | Hit Rate |
|-----------|-------|----------|
| #work     |   N   |    X%    |
| #personal |   N   |    X%    |

## Late Completions
- [task description] — scheduled 📅 MM-DD, done ✅ MM-DD (+N days)

## Misses
- [task description] — scheduled 📅 MM-DD, not completed

## Insight
[One sentence on what the data shows — pattern, trend, or recommendation]
```

### Insight Generation

Based on the data, provide ONE actionable observation:
- If hit rate >= 70%: Affirm the system is working, note any patterns
- If hit rate 50-69%: Identify where misses cluster (day of week, domain, etc.)
- If hit rate < 50%: Flag the system gap — are tasks too large? Days too fragmented?
- If late hits are high: Suggest the ONE thing is right but scoping may be off
- If misses cluster on specific days: Note the pattern

Mark Step 3 complete.

---

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| < 3 `🔺` tasks found | Output: "Insufficient data — need at least 3 ONE Big Thing entries for meaningful measurement. Found N so far." List the tasks found. |
| All tasks in same week | Show full data but add note: "Single-week sample — trend analysis not yet meaningful." |
| 0 `🔺` tasks found | Output: "No ONE Big Thing tasks found. Mark your top daily priority with 🔺 and a 📅 date to start tracking." |

## Output Guidelines

- **Screen output only.** Do not create or modify any vault files.
- **Be precise.** Show exact numbers, not just percentages.
- **Show the tasks.** Late completions and misses should list the actual task text so the user can see what slipped.
- **One insight, not a lecture.** Keep the analysis observation to one sentence.

## Integration

- `/weekly` — Can reference hit rate during weekly review
- `/monthly` — Include in monthly PKM effectiveness check
- `/close-day` — If today's `🔺` is incomplete at close, flag it
- `/daily` — Morning routine surfaces the current `🔺` task
