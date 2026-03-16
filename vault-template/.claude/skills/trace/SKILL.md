---
name: trace
description: "Trace how a specific idea, concept, or belief has evolved across the vault over time. Shows the full arc of thinking development. Screen output only."
allowed-tools: Read, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet
user-invocable: true
---

# Trace — Idea Evolution Over Time

Trace how a specific idea, concept, or belief has evolved across the vault over time. Shows the full arc of thinking development — from first appearance through inflection points to current position.

## Usage

```
/trace <topic>
```

Examples:
- `/trace account scoring`
- `/trace scope creep`
- `/trace AI integration`

## Obsidian CLI

All graph and search operations use the Obsidian CLI. The CLI outputs a loading line on startup — strip it:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian <command> 2>&1 | tail -n +3
```

Use this pattern for every CLI call throughout the workflow.

## Session Tasks (4 phases)

Create all 4 tasks upfront, then execute in order.

```
TaskCreate: subject="Discover vocabulary and search",  activeForm="Searching vault for topic mentions..."
TaskCreate: subject="Follow the graph",                activeForm="Tracing link connections..."
TaskCreate: subject="Build the timeline",              activeForm="Assembling chronological timeline..."
TaskCreate: subject="Identify the arc",                activeForm="Synthesizing evolution narrative..."
```

### Dependencies

```
TaskUpdate: "Follow the graph", addBlockedBy: [discover-id]
TaskUpdate: "Build the timeline", addBlockedBy: [graph-id]
TaskUpdate: "Identify the arc", addBlockedBy: [timeline-id]
```

All tasks are sequential.

---

## Step 1: Synonym Discovery & Search

**Goal:** Build a vocabulary map for the topic, then search exhaustively. Ideas often evolve under different vocabulary.

### 1a: Build Vocabulary Map

Read one context file most likely to touch this topic:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian read path="<most relevant Context/*.md file>" 2>&1 | tail -n +3
```

If no context file is obviously relevant, use Grep to find which files mention the topic:

```
Grep: pattern="<topic>" glob="*.md" output_mode="files_with_matches"
```

From the initial read, extract:
- 3-5 related terms, adjacent concepts, alternative phrasings
- Any jargon, shorthand, or metaphors used for this idea in the vault

### 1b: Search with Full Vocabulary

Search across the entire vault using the topic and all synonyms:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<topic>" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian search:context query="<topic>" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<synonym 1>" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<synonym 2>" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<related concept>" 2>&1 | tail -n +3
```

### 1c: Search Daily Notes Specifically

Daily notes capture thinking in real time — search them separately:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<topic>" path="Daily Notes" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<synonym 1>" path="Daily Notes" 2>&1 | tail -n +3
```

### 1d: Check Context Files, Essays, and MOCs

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<topic>" path="Context" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian search query="<topic>" path="General Notes" 2>&1 | tail -n +3
```

Read any context file likely to contain this topic in full.

### 1e: Sparse Data Handling

If the initial search returns few results, try broader related terms. If a topic has very few explicit mentions, it may appear implicitly. Move to Step 1.5.

### Step 1.5: Implicit Pattern Detection

Some ideas don't have explicit mentions but emerge through patterns across daily notes. Use broader contextual searches:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian search:context query="<broader theme>" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian tags counts sort=count 2>&1 | tail -n +3
```

Look for:
- Daily notes that discuss the same problem this idea addresses, without naming it
- Emotional reactions to situations that this idea would explain
- Decisions made that reflect this thinking, even if the thinking wasn't articulated

Implicit mentions are often the earliest evidence of an idea forming.

Mark Step 1 complete.

---

## Step 2: Follow the Graph

**Goal:** Trace connections outward from significant mentions to find related thinking, influences, and triggers.

For each note that mentions the topic:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian backlinks file="<note>" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian links file="<note>" 2>&1 | tail -n +3
```

Follow backlinks 2-3 hops out from the most significant mentions. The goal is to find:

- Notes that influenced this thinking without mentioning the topic directly
- Related concepts that evolved alongside this idea
- People, conversations, or events that triggered shifts

**Cap:** Do not follow more than 5-6 hub notes deep. Be selective — prioritize notes with dates and reflective content.

Mark Step 2 complete.

---

## Step 3: Build the Timeline

**Goal:** Organize findings chronologically, showing how the idea evolved over time.

For each significant mention or shift, record:

- **Date**: When it appeared
- **Context**: What was happening at the time (from daily notes, calendar if helpful)
- **The thinking**: What was believed or proposed at that point
- **Confidence level**: If marked with `[solid]`, `[evolving]`, `[hypothesis]`, `[questioning]`, note it
- **What triggered the shift**: Conversation, experience, reading, or just time

### Temporal Weighting

Recent mentions (past 3 months) suggest active evolution. The idea is alive and changing. Focus the narrative here. Older mentions establish origin but the story is in the active period.

If there's a long gap between mentions (e.g., nothing for 6 months, then a burst), that gap itself is interesting. What happened? Did the idea go dormant, or did it evolve underground?

Mark Step 3 complete.

---

## Step 4: Identify the Arc

**Goal:** Synthesize the timeline into a narrative that makes the shape of the user's thinking visible.

### First Appearance

When and where this idea first showed up. What form was it in? Was it a question, a hunch, a reaction to something?

### Key Inflection Points

Moments where the thinking shifted meaningfully. What caused each shift?

### Current Position

Where the thinking stands now. What confidence level? What's resolved vs. still open?

### Confidence Marker Evolution

If the topic appears in a context file with a confidence marker, show how that marker changed over time. Was it `[hypothesis]` and now `[solid]`? When did the shift happen? Was there a specific event that triggered the upgrade? Was it earned through experience, or did it drift to `[solid]` without being tested?

### The Evolution Pattern

What kind of evolution was this? Label with one of:

- **Linear deepening**: Same direction, just more refined over time
- **Pivots**: Fundamental changes in direction
- **Convergence**: Separate ideas that merged into one
- **Divergence**: One idea that split into multiple threads
- **Circular**: Keeps coming back to the same question without resolution

### Unresolved Tensions

Contradictions or open questions that remain. Things written at different times that don't agree with each other.

### What's Next

Based on the trajectory, where is this idea likely heading? What would resolve the open tensions?

Mark Step 4 complete.

---

## Output Format

```
TRACE: [Topic]
First appeared: [date]
Time span: [X weeks/months]
Mentions found: [number of notes]
Velocity: [Accelerating / Steady / Dormant]

---

## Timeline
[Chronological moments with dates, context, and quotes]

## Arc Narrative
[First appearance → Inflection points → Current position]

## Evolution Pattern
[Linear deepening / Pivots / Convergence / Divergence / Circular]

## Confidence Marker Evolution
[If applicable — how markers shifted over time]

## Unresolved Tensions
[Contradictions or open questions across time]

## What's Next
[Where the trajectory points, what would resolve tensions]
```

---

## Output Guidelines

- **Be specific:** Cite exact notes and dates
- **Show actual words:** Quote the language used at different times so the evolution is visible
- **Don't flatten complexity.** If the thinking contradicts itself across time, show that
- **Screen output only.** Do not create or modify any vault files
- The value is in seeing the shape of your own thinking from the outside. Make that shape visible.

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| Topic has 0 mentions | Report "No mentions found" with synonym suggestions |
| Topic has 1-2 mentions only | Produce abbreviated trace — skip graph following, focus on what exists |
| All mentions are within 7 days | Note "too recent for evolution analysis" — show current state instead |
| No context file touches the topic | Skip confidence marker analysis, note in output |

## Integration

Works with:
- `/ideas` — Trace can validate whether an idea from `/ideas` has been forming over time
- `/create-context` — Context files are primary sources for confidence markers and current state
- `/search` — Complementary: search finds mentions, trace shows how they evolved
- `/backlinks` — Graph structure from backlinks informs how ideas connect
- `/weekly` — Weekly reflections capture inflection points worth tracing
- `/close-day` — Daily reflections are primary source material for the timeline
