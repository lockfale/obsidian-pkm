---
name: backlinks
description: "Analyze vault link structure, find orphans, score missing connections, and wire the graph. Read-only analysis with optional interactive execution."
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, TaskGet
user-invocable: true
---

# Backlinks — Wire the Graph

Your vault is the context layer for an augmented cognition system. Every missing link is a capability the agent doesn't have. Every connection made compounds because the agent uses it in every future session. A connection that only existed in your head used to be fine. Now it's a capability gap that grows across every interaction.

**The boundary:** Your job is to make the graph traversable, not to generate understanding. You connect. The user thinks. When you find an empty hub, flag it — don't fill it. When you find notes that should link, wire them — don't rewrite them. The substance in this vault is human thinking. Your contribution is the wiring between those thoughts.

## Usage

```
/backlinks              # Full 7-phase vault graph analysis
/backlinks [cluster]    # Focus on a specific cluster — skips full scan, targets Phases 3-4
```

Or naturally: "Analyze my vault links" / "Find orphaned notes" / "Wire the graph"

If a cluster name is provided (e.g., `/backlinks defense-tech`), skip the full scan and focus Phases 3-4 on that cluster and its nearest neighbors. Faster results for known problem areas.

## Obsidian CLI

All graph operations use the Obsidian CLI. The CLI outputs a loading line on startup — strip it:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian <command> 2>&1 | tail -n +3
```

Use this pattern for every CLI call throughout the workflow.

## Session Task Tracking

Create all 7 tasks upfront at skill start:

```
TaskCreate: subject="Run structural inventory",          activeForm="Scanning vault link structure..."
TaskCreate: subject="Load priority context",             activeForm="Loading priorities and context..."
TaskCreate: subject="Rescue orphaned notes",             activeForm="Analyzing orphaned notes..."
TaskCreate: subject="Analyze cluster bridges",           activeForm="Finding cluster bridges..."
TaskCreate: subject="Triage unresolved links",           activeForm="Triaging unresolved links..."
TaskCreate: subject="Score and recommend connections",   activeForm="Scoring potential connections..."
TaskCreate: subject="Execute approved connections",      activeForm="Wiring approved connections..."
```

Set sequential dependencies — each task blocked by the previous one. Mark `in_progress` when starting, `completed` when done.

---

## Phase 1: Structural Inventory

Graph-only, fast. No content reads.

### Step 1a: Gather Graph Metrics

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian orphans 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian deadends 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian unresolved counts 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian tags counts sort=count 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian files total 2>&1 | tail -n +3
```

### Step 1b: Map Hub Notes

Get backlinks and outgoing links for the top 15-20 hub notes (notes that appear as frequent backlink targets):

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian backlinks file="<Hub Note>" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian links file="<Hub Note>" 2>&1 | tail -n +3
```

Build a rough cluster map from the link structure. Identify which notes are well-connected hubs vs isolated.

Mark Phase 1 complete.

---

## Phase 2: Priority Context

Read 5-6 context files to build a priority filter.

### Step 2a: Load Goal and Context Files

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian read path="Goals/1. Yearly Goals.md" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian read path="Goals/2. Monthly Goals.md" 2>&1 | tail -n +3
/Applications/Obsidian.app/Contents/MacOS/obsidian read path="Goals/3. Weekly Review.md" 2>&1 | tail -n +3
```

Also read any `Context/*.md` files that exist.

### Step 2b: Load Recent Daily Notes

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian daily:read 2>&1 | tail -n +3
```

Read daily notes from the past 7 days for active threads.

### Step 2c: Extract Priority Filter

From these reads, extract:

- Current priorities and active projects
- Open questions and active threads of thinking
- People and concepts getting current attention

This becomes the priority filter for scoring in Phase 6.

Mark Phase 2 complete.

---

## Phase 3: Orphan Rescue Scan

Filter orphans from Phase 1.

### Step 3a: Filter Non-Note Files

Skip non-markdown files:
- Images: `.heic`, `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`
- Media: `.mov`, `.mp4`, `.wav`, `.mp3`
- Fonts, canvases, templates
- Any file that isn't `.md`

### Step 3b: Triage Orphans

For remaining `.md` orphans, apply this decision tree:

1. **Title contains a word from current priorities or hub backlink chain?** Read title + first 20 lines. These are likely disconnected from active work.
2. **Title contains a question mark?** Read title + first 20 lines. These are often valuable thinking notes.
3. **Otherwise:** Read title only. Flag if semantically related to active clusters.

### Step 3c: Find Nearest Neighbors

For each meaningful orphan, search for its nearest cluster neighbor:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian search:context query="<key concept from orphan>" 2>&1 | tail -n +3
```

**Explicit cap:** Do not read more than 100 notes total across all phases. Be selective. Prioritize notes whose titles suggest relevance to current priorities.

Mark Phase 3 complete.

---

## Phase 4: Cluster Bridge Analysis

For each pair of the top 5-6 clusters with no existing links, search for shared themes.

### Step 4a: Cross-Cluster Search

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian search:context query="<theme from Cluster A>" 2>&1 | tail -n +3
```

Search in Cluster B's territory for Cluster A themes, and vice versa.

### Step 4b: Read Bridge Candidates

Read intermediary notes that might serve as bridges between clusters.

### Step 4c: Vocabulary Variants

Use synonym discovery: try 2-3 vocabulary variants for each concept. Ideas evolve under different names. A concept called "threat modeling" in one cluster might appear as "risk assessment" in another.

Mark Phase 4 complete.

---

## Phase 5: Unresolved Link Triage

From the unresolved links gathered in Phase 1:

| References | Action |
|------------|--------|
| 3+ references | Likely worth creating as a stub note |
| 2 references | Check for near-duplicates or name variants of existing notes |
| 1 reference | Skip unless it's a critical concept from current priorities |

Check for name variants that resolve to existing notes before recommending new stubs.

Mark Phase 5 complete.

---

## Phase 6: Score and Recommend

Score each candidate connection on 3 dimensions (multiplicative):

| Dimension | What it measures | 1 | 3 | 5 |
|-----------|-----------------|---|---|---|
| Conceptual Strength | How real is this connection? | Same word, different context | Related problems/questions | Same thesis from different angles |
| Structural Impact | How much does this improve the graph? | 6th link to well-connected note | Rescues valuable orphan | Bridges two isolated clusters |
| Priority Alignment | Does it touch current priorities? | Neither note is active | One note relates to a priority | Both notes relate to active work |

- Composite = Conceptual x Structural x Priority (max 125)
- **Critical** (75+), **High** (40-74), **Medium** (15-39), **Low** (<15)

### Quality Controls

- Minimum Conceptual Strength of 2. Same word but different concept = skip.
- Cap at 30 total recommendations. Quality over quantity.
- Each connection must be explainable in one sentence.
- Include borderline cases with lower scores rather than silently excluding.

### Connection Taxonomy

Label each recommendation with one of these types:

1. **Orphan to Hub** — Orphaned note connected to its nearest cluster hub
2. **Cluster Bridge** — Two clusters share themes but zero links between them
3. **Internal Gap** — Within-cluster notes missing cross-references
4. **Empty Hub (flag only)** — Hub note referenced by many but has no content. Flagged for the user to write. NOT filled by agent.
5. **Semantic Twin** — Two notes about the same concept, different vocabulary
6. **Person to Concept** — Person note that should link to associated concepts
7. **Temporal Bridge** — Old thinking relevant to new work, never connected
8. **Unresolved Worth Creating** — `[[link]]` appearing 3+ times, worth formalizing

### Connection Card Format

For each recommendation:

```
### [#]. [Type]: [Short description]
**Score:** [X] (Conceptual [N] x Structural [N] x Priority [N])
**What:** [One-sentence description]
**Edit:** Add `[[Target Note]]` to [Source Note] in [section/location]
**Why:** [One sentence on what this unlocks]
```

For Empty Hub flags, use this format instead:

```
### [#]. Empty Hub: [Note name]
**Referenced by:** [list of notes that link to it]
**What this needs:** Your thinking. [N] notes point here and find nothing.
```

Mark Phase 6 complete.

---

## Phase 7: Execute

Present findings by tier (Critical first).

### Step 7a: Ask for Execution Scope

```
AskUserQuestion:
  question: "Which connections should I execute?"
  options:
    - label: "All Critical + High (Recommended)"
      description: "Execute all Critical and High tier connections"
    - label: "Critical only"
      description: "Execute only Critical tier connections"
    - label: "Pick specific ones"
      description: "Choose individual connections by number"
    - label: "None — save report only"
      description: "Keep the analysis without making changes"
```

### Step 7b: Execution Rules

When executing approved connections:

- **Adding `[[links]]`:** Place in the relevant section of the note, not dumped at the bottom. Read the note structure first and add the link where it makes semantic sense.
- **Creating stub notes** (for Unresolved Worth Creating): Title + a "Related" section listing backlinks ONLY. No synthesized content. No descriptions. No summaries.
- **Empty Hubs:** DO NOT fill. They appear in the report as flags only. The user writes the content.

### Step 7c: Post-Execution Verification

After making changes, verify links resolved:

```bash
/Applications/Obsidian.app/Contents/MacOS/obsidian links file="<modified note>" 2>&1 | tail -n +3
```

Report: "Made X connections across Y notes. Z new stub notes created."

Mark Phase 7 complete.

---

## Output Format

```
BACKLINKS REPORT -- [Date]
Vault size: [N] notes
Orphans assessed: [N] meaningful (of [N] total)
Connections recommended: [N] across [N] tiers

---
## Critical ([N])
[Connection cards]

## High ([N])
[Connection cards]

## Medium ([N])
[Connection cards]

## Summary
[Narrative: biggest structural gaps, what executing these changes unlocks]

---
Execute: All Critical+High / Critical only / Pick specific / None
```

---

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| No orphans found | Skip Phase 3, note in report |
| No unresolved links | Skip Phase 5, note in report |
| Vault has < 10 notes | Report vault too small for meaningful graph analysis |
| Cluster argument not found | List available clusters from Phase 1, ask user to pick |
| Cluster argument provided | Skip full scan, focus Phases 3-4 on that cluster |
| All connections score Low | Report "graph is well-connected" with any Low suggestions |
| User selects "None" in Phase 7 | Save report output only, no file modifications |

## Limitations

Be honest about these:

- **False positives:** Include with lower score rather than silently exclude. Let the user decide.
- **Vocabulary drift:** Try 2-3 variants for each concept. Ideas evolve under different names.
- **Recency bias:** Flag old-but-rich orphans even if they don't match current priorities. Old thinking can be the most valuable.
- **Graph vs. content tension:** Well-connected notes can have shallow content. Poorly-connected notes can be deeply thoughtful. Connection count alone doesn't indicate value.

## Conventions

This skill follows all CLAUDE.md conventions:

- **Wiki-links:** Use `[[Note Name]]` format for all vault references
- **Frontmatter:** Include YAML frontmatter on all created stub notes
- **Heading structure:** H1 for title, H2 for sections, H3 for subsections
- **Link placement:** Semantic location within notes, never dumped at bottom
- **Empty hubs:** Flag only — never fill with generated content

## Integration

Works with:
- `/check-links` — Find broken links (complementary: check-links finds broken refs, backlinks finds missing connections)
- `/close-day` — Evening processing may surface orphaned captures worth connecting
- `/create-context` — Context files inform priority scoring; backlinks may reveal context gaps
- `/daily` — Daily notes provide recency signals for priority alignment scoring
- `/weekly` — Weekly review surfaces active threads that inform cluster priorities
