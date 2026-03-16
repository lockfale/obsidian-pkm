---
name: challenge
description: "Push back on current thinking by surfacing contradictions, unearned confidence, and stale beliefs using vault history. Screen output only."
allowed-tools: Read, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet, AskUserQuestion
user-invocable: true
---

# Challenge — Push Back on My Thinking

Use the vault's own history to challenge current beliefs, surface contradictions, and pressure-test evolving ideas. This is not about being contrarian. It's about using the accumulated record of thinking to find where the cracks are.

## Usage

```
/challenge                    # General — scan all active beliefs
/challenge <topic>            # Focused — challenge thinking on a specific topic
```

Examples:
- `/challenge` — general challenge across all active context
- `/challenge scope creep`
- `/challenge AI integration`
- `/challenge leadership style`

## Session Tasks (4 phases)

Create all 4 tasks upfront, then execute in order.

```
TaskCreate: subject="Identify beliefs to challenge",   activeForm="Scanning vault for active beliefs..."
TaskCreate: subject="Find contradictions",              activeForm="Searching for contradicting evidence..."
TaskCreate: subject="Build challenges",                 activeForm="Structuring challenge report..."
TaskCreate: subject="Synthesize patterns and actions",  activeForm="Synthesizing meta-patterns..."
```

### Dependencies

```
TaskUpdate: "Find contradictions", addBlockedBy: [identify-id]
TaskUpdate: "Build challenges", addBlockedBy: [contradictions-id]
TaskUpdate: "Synthesize patterns and actions", addBlockedBy: [challenges-id]
```

All tasks are sequential.

---

## Step 1: Identify What to Challenge

**Goal:** Surface the beliefs and positions to examine — either broadly from vault context, or narrowly from a specified topic.

### General Path (no topic provided)

#### 1a: Read Context Files

```
Glob: pattern="Context/**/*.md"
```

Read all context files. Extract items marked with confidence markers:
- `[evolving]` — actively changing, worth pressure-testing
- `[hypothesis]` — untested, may be wishful thinking
- `[questioning]` — already flagged as uncertain

Also extract any position stated with high confidence (`[solid]`) that hasn't been revisited recently.

#### 1b: Read Recent Daily Notes

```
Glob: pattern="Daily Notes/YYYY-MM-DD.md"  # past 7-14 days
```

Read each note. Focus on Capture, Notes from Today, and Reflection sections. Skip `tasks` query blocks.

#### 1c: Structured Belief Extraction

For each daily note and context file, extract beliefs explicitly. Look for statements containing:

- "I believe...", "I think...", "the issue is...", "what matters is..."
- "The way to...", "The problem with...", "What works is..."
- Strong assertions, confident predictions, or absolute statements
- Positions presented as settled that may not have been tested

List these as raw statements with their source (`[[note name]]` or date) before proceeding. This prevents vague "the vault feels contradictory" analysis and forces specificity.

### Focused Path (topic provided)

#### 1d: Search for Topic

```
Grep: pattern="<topic>" glob="**/*.md" output_mode="files_with_matches"
```

Read the most relevant matches. Also search for synonyms, related terms, and alternative phrasings the vault may use.

#### 1e: Extract Topic-Specific Beliefs

From the search results, extract all positions, claims, and assertions related to the topic. Include the source and date for each.

### Sparse Data Handling

| Condition | Behavior |
|-----------|----------|
| General path finds < 3 beliefs | Expand daily note window to 30 days |
| Focused path finds 0 mentions | Report "No mentions found" with synonym suggestions |
| Focused path finds 1-2 mentions | Produce abbreviated challenge — work with what exists |

Mark Step 1 complete.

---

## Step 2: Find the Contradictions

**Goal:** For each identified belief, search the vault for evidence that contradicts, complicates, or undermines it.

### 2a: Search for Contradicting Evidence

For each belief from Step 1:

```
Grep: pattern="<opposite or complicating concept>" glob="**/*.md"
Grep: pattern="<earlier version of this thinking>" path="Daily Notes"
```

Search for:
- **Self-contradictions**: Places where something was written that directly conflicts with a current belief
- **Abandoned positions**: Things once held strongly but quietly dropped — why?
- **Unearned confidence**: Things marked `[solid]` that were never actually tested or proven
- **Wishful thinking**: Hypotheses treated as conclusions without evidence
- **Blind spots**: Important questions never asked; adjacent topics never explored
- **Stale thinking**: Positions not updated despite new information or experience

### 2b: Temporal Weighting

A belief contradicted 18 months ago has different weight than one contradicted last month.

- **Recent contradictions** (past 3 months): Active confusion. These need resolution now.
- **Medium-term contradictions** (3-12 months): May represent genuine evolution. Check if the old position was consciously abandoned or just forgotten.
- **Old contradictions** (12+ months): Likely represent growth, not confusion. Only flag if the old position was never explicitly updated.

Flag only active tensions. Don't surface ancient contradictions as current problems.

### 2c: Cross-Context Comparison

After individual analysis, explicitly check: do different context files stake contradictory positions on the same topic?

```
Grep: pattern="<belief statement keywords>" path="Context"
```

Cross-file tensions are often the most important — they indicate compartmentalized thinking where something is true in one domain but denied in another.

Mark Step 2 complete.

---

## Step 3: Build the Challenges

**Goal:** Structure each finding into a clear, actionable challenge with evidence and severity.

### Challenge Format

For each challenge:

```markdown
### Challenge [#]: [Short title]

**Current position:** [What is currently believed, with citation to specific note/date]

**The problem:** [What contradicts, complicates, or undermines this]

**Evidence from the vault:** [Specific notes, dates, and quotes that create the tension]

**The question this raises:** [A genuine question — not rhetorical — that would need answering to resolve the tension]

**Severity:** [Crack / Tension / Foundation risk]
```

### Severity Levels

- **Crack**: Minor inconsistency, worth noting but not urgent
- **Tension**: Real contradiction that needs addressing
- **Foundation risk**: If this is wrong, a lot of other thinking built on top of it is also wrong

### Ordering

Present challenges ordered by severity (Foundation risk first, Cracks last).

Mark Step 3 complete.

---

## Step 4: Synthesize

**Goal:** Step back from individual challenges to find meta-patterns, surface the hardest question, and recommend concrete actions.

### 4a: Patterns in the Challenges

What do the challenges have in common? Look for a meta-pattern, e.g.:
- "Most evolving ideas haven't been tested against reality"
- "You keep theorizing about X without talking to anyone who's done it"
- "Confidence levels are climbing without new evidence"

### 4b: The Hardest Question

What's the single most uncomfortable question the vault raises about current thinking? The one that would be easiest to avoid and most valuable to face.

### 4c: Recommended Actions

For each challenge, suggest one concrete action that would resolve the tension:

- A conversation to have
- An experiment to run
- A note to write that works through the contradiction
- A belief to update
- A question to sit with longer

Mark Step 4 complete.

---

## Output Format

```
CHALLENGE REPORT
Scope: [General / focused on <topic>]
Beliefs examined: [number]
Challenges found: [number]

---

[Challenges, ordered by severity — Foundation risk → Tension → Crack]

---

## Patterns
[Meta-patterns across the challenges]

## The Hardest Question
[Single most uncomfortable question]

## Recommended Actions
[One concrete action per challenge]
```

---

## Output Guidelines

- **Be honest, not harsh.** The goal is clarity, not criticism.
- **Always cite specific notes and dates.** Vague challenges are useless.
- **The best challenges make you think "I hadn't noticed that"**, not "that's unfair."
- **Don't challenge things marked `[solid]`** unless the vault genuinely contains contradicting evidence.
- **Screen output only.** Do not create or modify any vault files. The user copies what they want.
- This should feel like talking to the most honest, well-informed version of yourself.

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| No context files and < 7 daily notes | Report "Insufficient vault data for meaningful challenge" |
| Topic has 0 mentions | Report "No mentions found" with synonym suggestions |
| No contradictions found for any belief | Report honestly — note which beliefs were examined and that they appear internally consistent |

## Integration

Works with:
- `/trace` — Trace the evolution of a challenged belief
- `/ideas` — Challenge results may spark new ideas worth exploring
- `/create-context` — Context files are primary input (confidence markers drive belief extraction)
- `/weekly` — Challenges feed into weekly reflection
- `/close-day` — Challenge insights can inform daily reflection
- Goal tracking — Foundation risks may affect goal assumptions
