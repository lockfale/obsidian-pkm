---
name: contradict
description: "Find incompatible beliefs held simultaneously in the vault. Not evolution (trace), not pressure-testing (challenge). Finds beliefs that cannot both be true right now. Screen output only."
allowed-tools: Read, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet, AskUserQuestion
user-invocable: true
---

# Contradict — Find Incompatible Beliefs Held Simultaneously

Surface beliefs that cannot both be true at the same time. Not evolution (that's `/trace`), not pressure-testing (that's `/challenge`). This command finds places where the vault holds two positions that are logically incompatible, right now, and asks: which one do you actually believe?

## Usage

```
/contradict                    # General — scan all active beliefs
/contradict <domain>           # Focused — contradictions within a specific domain
```

Examples:
- `/contradict` — general scan across all context and recent notes
- `/contradict leadership`
- `/contradict AI integration`
- `/contradict priorities`

## How This Differs from /challenge and /trace

- **/challenge** pressure-tests current beliefs by finding evidence against them. It asks: "Is this wrong?"
- **/trace** tracks how ideas evolved over time. It asks: "How did this change?"
- **/contradict** finds beliefs held simultaneously that cannot cohere. It asks: "You believe X and also Y. Both cannot be true. Which is it?"

A challenge might reveal a belief is weak. A trace might show a belief changed. A contradiction reveals that two beliefs currently coexist and shouldn't.

## Session Tasks (5 phases)

Create all 5 tasks upfront, then execute in order.

```
TaskCreate: subject="Extract beliefs",              activeForm="Extracting beliefs from vault..."
TaskCreate: subject="Detect contradictions",         activeForm="Comparing beliefs for logical conflicts..."
TaskCreate: subject="Filter through evolution tests", activeForm="Filtering out resolved contradictions..."
TaskCreate: subject="Classify contradictions",       activeForm="Classifying by category and weight..."
TaskCreate: subject="Synthesize findings",           activeForm="Building contradiction report..."
```

### Dependencies

```
TaskUpdate: "Detect contradictions", addBlockedBy: [extract-id]
TaskUpdate: "Filter through evolution tests", addBlockedBy: [detect-id]
TaskUpdate: "Classify contradictions", addBlockedBy: [filter-id]
TaskUpdate: "Synthesize findings", addBlockedBy: [classify-id]
```

All tasks are sequential.

---

## Step 1: Extract Beliefs

**Goal:** Build a working list of at least 15-20 explicit belief statements from the vault. Each belief needs its source and implied inverse.

### 1a: Read Context Files

```
Glob: pattern="Context/**/*.md"
```

Read all context files. These are the richest source of beliefs — especially items marked `[solid]`, `[evolving]`, or `[hypothesis]`.

### 1b: Read Recent Daily Notes

```
Glob: pattern="Daily Notes/YYYY-MM-DD.md"  # past 14-21 days
```

Read each note. Focus on Capture, Notes from Today, Reflection, and Ideas sections. Skip `tasks` query blocks.

### 1c: Read Goal Files

```
Read: Goals/0. Three Year Goals.md
Read: Goals/1. Yearly Goals.md
Read: Goals/2. Monthly Goals.md
Read: Goals/3. Weekly Review.md
```

Goals contain implicit beliefs about what matters, what's achievable, and how to spend time.

### Focused Path (domain provided)

If a domain is specified, narrow scope:

```
Grep: pattern="<domain>" glob="**/*.md" output_mode="files_with_matches"
```

Read the most relevant matches. Also search for synonyms and related terms.

### 1d: Structured Belief Extraction

For each source, extract explicit belief statements. Look for:

- "I believe...", "I think...", "The way to...", "What matters is..."
- Strong assertions, confident predictions, absolute statements
- Advice given to others (implies a belief)
- Decisions made (imply a belief about what's right)
- Priorities stated (imply a belief about what matters)
- Things rejected or criticized (imply a belief about what's wrong)

**Write each belief as a paired statement:**

- **Belief:** "[Explicit statement]"
- **Source:** [Note name, date, context]
- **Implied inverse:** "[What this belief denies or excludes]"

The implied inverse is critical — it's what makes contradiction detection possible. "I believe in focus" implies "spreading thin is wrong." If another part of the vault shows enthusiastic spreading thin, that's a contradiction.

**List ALL extracted beliefs before moving to detection. Minimum 15-20 beliefs to work with.**

### Sparse Data Handling

| Condition | Behavior |
|-----------|----------|
| General path finds < 5 beliefs | Expand daily note window to 30 days |
| Focused path finds 0 mentions | Report "No mentions found" with synonym suggestions |
| Focused path finds 1-3 mentions | Produce abbreviated analysis — work with what exists |

Mark Step 1 complete.

---

## Step 2: Detect Contradictions

**Goal:** Run three independent detection methods. Each catches contradictions the others miss.

### Method 1: Explicit Contradictions

#### Within-Domain

Compare beliefs extracted from the same context file or same time period. Do any directly conflict?

```
Grep: pattern="<belief A keywords>" glob="Context/**/*.md"
Grep: pattern="<belief A inverse keywords>" glob="Context/**/*.md"
```

#### Cross-Domain

Compare beliefs across different context files. A belief about building may contradict a belief about living. These are the most interesting contradictions because they reveal compartmentalized thinking.

```
Grep: pattern="<belief from domain A keywords>" glob="Context/<domain B>*"
```

### Method 2: Implicit Contradictions

#### Stated Values vs. Revealed Preferences

What does the vault say matters vs. what does actual behavior (daily notes, time allocation, energy levels) reveal? A stated value of "deep focus" alongside daily notes filled with reactive scattered activity is an implicit contradiction.

```
Grep: pattern="<stated value>" glob="Daily Notes/**/*.md"
```

#### Advice vs. Behavior

What advice does the vault give to others vs. what does the vault show about personal behavior? Advice given that isn't followed is a contradiction.

#### Decisions vs. Stated Beliefs

Do decisions recorded in project files or daily notes align with the beliefs stated in context files?

### Method 3: Priority Contradictions

Two things cannot both be the top priority. If the vault treats multiple things as most important, that's a priority contradiction. These are often hidden because each domain has its own context file where it gets to be the center of attention.

Compare priority language across all context files and goal files. How many things are described as "the most important" or "the priority" or "what matters most"?

Mark Step 2 complete.

---

## Step 3: Evolution Filter

**Goal:** Before reporting any contradiction, run it through this 4-test filter. Many apparent contradictions are actually evidence of healthy evolution.

### Test 1: Temporal Currency

Are both beliefs current? If one was written 6+ months ago and the other last week, this might be evolution, not contradiction.

```
Grep: pattern="<older belief keywords>" glob="Daily Notes/**/*.md"
```

If the older belief hasn't appeared in 3+ months, it may simply be outdated. Flag as "possibly resolved through evolution" rather than an active contradiction.

### Test 2: Domain Compartmentalization

Is it possible that both beliefs are true in their respective domains? "Move fast" in product and "be patient" in relationships is not a contradiction. But "move fast" in product and "be patient" in product IS one. Only flag cross-domain contradictions when the beliefs genuinely cannot coexist.

### Test 3: Confidence Marker Check

Do the contradicting beliefs have confidence markers? A `[solid]` belief contradicting a `[hypothesis]` is less concerning than two `[solid]` beliefs contradicting each other.

### Test 4: The "Cannot Both Be True" Test

The hardest test. State both beliefs plainly and ask: is there any reasonable interpretation where both can be true simultaneously? If yes, this is tension, not contradiction. Only report contradictions that genuinely fail this test.

**Only contradictions that survive all 4 tests proceed to Step 4.**

Mark Step 3 complete.

---

## Step 4: Classify Each Contradiction

**Goal:** Assign a category and weight to each surviving contradiction.

### Category

**Resolve**: Confused thinking. One of these beliefs needs to go. The contradiction is producing real costs (bad decisions, wasted energy, incoherent strategy).

**Hold**: Productive tension. The contradiction is actually useful because reality is complex. Both beliefs capture something true, and holding them in tension is better than collapsing into one. But name it explicitly so it's held consciously, not accidentally.

**Celebrate**: Evidence of growth. An old belief and a new belief that contradict each other, where the new one represents genuine learning. The contradiction exists because the vault hasn't been updated, not because thinking is confused.

### Weight

| Weight | Definition |
|---|---|
| **Trivial** | Interesting but inconsequential. Different moods, different contexts. |
| **Moderate** | Affects decision-making in a specific area. Worth noting. |
| **Significant** | Affects strategy or identity. Needs conscious attention. |
| **Foundational** | If unresolved, undermines the coherence of a major life area. |

Mark Step 4 complete.

---

## Step 5: Output

### For Each Contradiction

```markdown
### Contradiction [#]: [Title]

**Belief A:** "[Statement]"
- Source: [Note, date]
- Confidence: [If marked]

**Belief B:** "[Statement]"
- Source: [Note, date]
- Confidence: [If marked]

**Why these cannot both be true:** [Explicit logical statement]

**Category:** Resolve / Hold / Celebrate
**Weight:** Trivial / Moderate / Significant / Foundational

**What to do:** [Specific recommendation based on category]
```

### Synthesis

```markdown
## Summary

**Contradiction count:** [number]
**By category:** Resolve: X, Hold: Y, Celebrate: Z
**By weight:** Foundational: X, Significant: Y, Moderate: Z, Trivial: W

## The Deepest Contradiction
[The one that, if resolved, would clarify the most about everything else.]

## Pattern in the Contradictions
[Do they cluster? Is there a meta-contradiction — e.g., "Most contradictions are between what you say and what you do" or "Most contradictions are between your ambitions and your constraints"?]
```

### Full Output Format

```
CONTRADICT REPORT
Scope: [General / Domain-focused: <domain>]
Beliefs examined: [number]
Contradictions detected: [number]
Survived evolution filter: [number]

---

[Contradictions, ordered by weight — Foundational first, Trivial last]

---

[Summary]
[The Deepest Contradiction]
[Pattern Analysis]
```

Mark Step 5 complete.

---

## Anti-Patterns

**1. The Evolution Trap**
Flagging changed beliefs as contradictions. If someone believed X in January and Y in June, that's evolution. Only flag it if BOTH are still active.

**2. The Nuance Destroyer**
Treating domain-appropriate variation as contradiction. Different contexts can warrant different approaches. Only flag when beliefs genuinely cannot coexist.

**3. The Contradiction Factory**
Finding contradictions everywhere by paraphrasing beliefs loosely enough that anything conflicts with anything. Be precise about what each belief actually claims.

**4. The Problem Frame**
Treating all contradictions as problems. Some contradictions are productive tensions that should be held, not resolved.

**5. The Surface Scanner**
Only finding explicit contradictions (word-level conflicts) and missing implicit ones (value-behavior gaps, advice-action gaps). Method 2 is where the real findings are.

**6. The Therapist**
Turning contradiction analysis into psychological diagnosis. This is logic analysis, not therapy. Stay structural.

---

## Output Guidelines

- **Be precise.** Vague contradictions are not contradictions, they're mood shifts.
- **Always cite specific notes and dates.** Every belief should be traceable.
- **The evolution filter is mandatory.** Skipping it produces false positives.
- **The best contradictions** make you say "I genuinely believe both of those, and you're right, I can't."
- **If the vault is genuinely non-contradictory, say so.** Don't manufacture contradictions.
- **Screen output only.** Do not create or modify any vault files. The user copies what they want.

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| No context files and < 7 daily notes | Report "Insufficient vault data for meaningful contradiction analysis" |
| Domain has 0 mentions | Report "No mentions found" with synonym suggestions |
| No contradictions survive evolution filter | Report honestly — list beliefs examined and note they appear internally consistent |

## Integration

Works with:
- `/challenge` — Challenge pressure-tests individual beliefs; Contradict finds pairs that cannot coexist
- `/trace` — Trace the evolution of a contradicting belief to understand how it formed
- `/drift` — Drift surfaces intention-vs-behavior gaps; Contradict surfaces belief-vs-belief gaps
- `/ideas` — Contradiction resolution may spark new synthesis ideas
- `/create-context` — Context files with confidence markers are primary input
- `/weekly` — Contradictions feed into weekly reflection
- `/close-day` — Contradiction insights can inform daily reflection
