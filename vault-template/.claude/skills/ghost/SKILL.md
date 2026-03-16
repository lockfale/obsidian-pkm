---
name: ghost
description: "Answer a question as the vault's author would, drawing only on vault content, then evaluate fidelity. Screen output only."
allowed-tools: Read, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet, AskUserQuestion
user-invocable: true
---

# Ghost -- Answer a Question as You from the Vault

Take a question and answer it the way the vault's author would, drawing only on what the vault contains. Then pull back the curtain and honestly evaluate how faithful the answer is.

## Usage

```
/ghost [question]
```

Examples:
- `/ghost What do you think about AI wrappers?`
- `/ghost What advice would you give a first-time founder?`
- `/ghost How do you feel about remote work?`
- `/ghost What will happen with LLMs in the next 2 years?`

## Session Tasks (5 phases)

Create all 5 tasks upfront, then execute in order.

```
TaskCreate: subject="Parse question type",    activeForm="Classifying question and setting search strategy..."
TaskCreate: subject="Build voice profile",    activeForm="Extracting author voice from vault..."
TaskCreate: subject="Gather evidence",        activeForm="Searching vault for relevant positions and beliefs..."
TaskCreate: subject="Compose ghost answer",   activeForm="Writing answer in author's voice..."
TaskCreate: subject="Behind the curtain",     activeForm="Evaluating fidelity and building evidence map..."
```

### Dependencies

```
TaskUpdate: "Build voice profile", addBlockedBy: [parse-id]
TaskUpdate: "Gather evidence", addBlockedBy: [voice-id]
TaskUpdate: "Compose ghost answer", addBlockedBy: [evidence-id]
TaskUpdate: "Behind the curtain", addBlockedBy: [compose-id]
```

All tasks are sequential.

---

## Phase 1: Parse Question Type

**Goal:** Classify the question before searching. The type determines how much inference is acceptable and which search strategy to use.

| Type | Description | Search Strategy | Inference Tolerance |
|------|-------------|-----------------|---------------------|
| **Factual** | "What is your company structure?" | Direct search for facts | Very low. Vault must contain the answer. |
| **Opinion** | "What do you think about X?" | Search for stated positions, adjacent beliefs | Moderate. Can infer from patterns if explicit opinion absent. |
| **Advice** | "What would you tell someone about X?" | Search for beliefs, experiences, lessons learned | Moderate-high. Can synthesize from multiple sources. |
| **Personal** | "How do you feel about X?" | Daily notes, emotional language, repeated themes | Low. Don't fabricate feelings. |
| **Prediction** | "What do you think will happen with X?" | Search for stated predictions, trend analysis, beliefs about the future | High, but must be grounded in vault worldview. |

Record the question type before proceeding. It gates everything that follows.

### Sparse Data Handling

| Condition | Behavior |
|-----------|----------|
| Question is too vague to classify | Ask user to rephrase with more specificity |
| Question touches multiple types | Pick the dominant type, note the secondary |

Mark Phase 1 complete.

---

## Phase 2: Build Voice Profile

**Goal:** Extract the author's voice from the vault before writing a single word. This prevents the answer from defaulting to Claude's voice with the user's opinions pasted in.

### 2a: Read Source Material

```
Bash: date math to determine past 14 days of daily note filenames
Glob: pattern="Daily Notes/YYYY-MM-*.md"   # recent daily notes
```

Read the past 14 days of daily notes. Also read:

```
Glob: pattern="Context/**/*.md"
```

Read context files and any general notes that show the author's natural writing voice (reflections, essays, long-form captures).

### 2b: Extract Voice Characteristics

From the material read, explicitly note these before composing:

1. **Sentence structure**: Short and declarative? Long and exploratory? Mixed?
2. **Vocabulary**: What words do they reach for? What words do they never use?
3. **Rhetorical patterns**: Do they use analogies? Lists? Questions? Provocations?
4. **Emotional register**: Direct and dry? Passionate? Measured? When does it shift?
5. **Characteristic phrases**: Specific turns of phrase, recurring metaphors, verbal tics
6. **What they do NOT do**: What stylistic choices are conspicuously absent?

This voice profile is a hard constraint on the ghost answer. If the answer doesn't sound like these notes, it fails.

### Sparse Data Handling

| Condition | Behavior |
|-----------|----------|
| < 5 daily notes available | Expand window to 30 days; lean on context files and general notes |
| No context files exist | Build voice profile from daily notes and goal files only; note reduced confidence |
| Very little prose in vault (mostly tasks) | Note that voice profile is weak; flag in fidelity assessment |

Mark Phase 2 complete.

---

## Phase 3: Gather Evidence

**Goal:** Find everything the vault contains that's relevant to the question, tag it by source quality, and list it before composing.

### 3a: Synonym Discovery

Before searching, build a vocabulary map for the question's topic:

1. Read one context file or general note most likely to touch the topic
2. Extract 3-5 related terms, adjacent concepts, alternative phrasings
3. Note any shorthand, jargon, or metaphors used for this idea in the vault

### 3b: Multi-Method Search

```
Grep: pattern="<topic>" glob="**/*.md"
Grep: pattern="<synonym 1>" glob="**/*.md"
Grep: pattern="<synonym 2>" glob="**/*.md"
Grep: pattern="<topic>" glob="Daily Notes/*.md"
Grep: pattern="<topic>" glob="Context/**/*.md"
Grep: pattern="<note title>" glob="**/*.md"   # backlink search for most relevant note
```

Read the most relevant files found. Follow links and backlinks from the most relevant notes.

### 3c: Evidence Tagging

For every piece of relevant information found, tag it immediately:

- **VAULT**: The user explicitly said or wrote this. Direct quote available.
- **INFERRED**: Not explicitly stated, but strongly implied by 2+ vault sources.
- **EXTRAPOLATED**: Consistent with their worldview but requires extending beyond what's written.
- **UNKNOWN**: The vault contains nothing relevant to this aspect of the question.

**List ALL tagged evidence before composing. Do not skip this step.**

### Sparse Data Handling

| Condition | Behavior |
|-----------|----------|
| Primary topic search returns < 3 results | Broaden with synonym search and related concepts |
| Still thin after broadening | Check if the topic appears implicitly (decisions or actions that reflect a position without stating it) |
| Check backlinks 2-3 hops from related notes | Follow the link graph for indirect connections |
| Still thin after all methods | This is critical information for the fidelity score. Do not compensate by inventing material. |

Mark Phase 3 complete.

---

## Phase 4: Compose Ghost Answer

**Goal:** Write the answer as the vault's author would, with inline source tags and hard thresholds on extrapolation.

### Constraints

1. **Use the voice profile from Phase 2.** Every sentence should pass the test: "Would they actually say it this way?"
2. **Inline source tags.** After each claim or position, note the source tag in brackets: `[VAULT: note name]`, `[INFERRED: from X + Y]`, `[EXTRAPOLATED: based on Z]`, `[UNKNOWN]`.
3. **Hard thresholds:**
   - If >30% of the answer is EXTRAPOLATED: add an explicit caveat at the top: *"This answer leans heavily on extrapolation. The vault doesn't contain enough to answer this with high confidence."*
   - If >50% is EXTRAPOLATED or UNKNOWN: **decline to give a ghost answer.** Instead, report what the vault DOES contain and where the gaps are.
4. **Length and tone**: Match what the author would actually produce. If they'd give a two-sentence answer, don't write five paragraphs. If they'd go deep, go deep.

### Anti-Patterns (check each before finalizing)

**1. The Wikipedia Ghost**
Writing a neutral, encyclopedic answer about the topic instead of the user's actual position. Ghost answers should be opinionated where the vault shows opinions.

**2. The Confident Fabricator**
Generating a fluent, confident answer that sounds plausible but isn't grounded in vault evidence. The smoothness of the answer masks the absence of real data. If the vault is thin, the answer should be thin.

**3. The Claude Ghost**
Using Claude's default voice, vocabulary, and rhetorical style instead of the user's. Watch for: bullet-point-heavy structure, "it's worth noting," "let's unpack," hedging qualifiers, and balanced-both-sides framing that the user wouldn't use.

**4. The Therapist Ghost**
Turning the answer into self-help wisdom or reflective questions instead of giving the direct, sometimes blunt answer the user would give.

**5. The Yes Ghost**
Assuming the user would agree with the premise of the question. If the vault shows they would push back on the question itself, the ghost should push back.

**6. The Safe Ghost**
Softening positions the vault shows are held strongly. If the user is direct or provocative about something, the ghost answer should be too.

Mark Phase 4 complete.

---

## Phase 5: Behind the Curtain

**Goal:** Honestly evaluate the ghost answer's fidelity. This section is not optional -- it's the whole point.

### Evidence Map

| Claim | Source Tag | Source Note(s) | Confidence |
|-------|-----------|----------------|------------|
| ... | VAULT | Note name, date | High |
| ... | INFERRED | Note A + Note B | Medium |
| ... | EXTRAPOLATED | Based on pattern in X | Low |

### Source Distribution

- VAULT: X%
- INFERRED: X%
- EXTRAPOLATED: X%
- UNKNOWN: X%

### Vault Gaps

What would the vault need to contain to answer this question with full confidence? Be specific: "A note about experience with X" or "Any mention of how they feel about Y."

### Voice Fidelity Notes

Where did the ghost answer match their voice well? Where did it slip into Claude's default patterns? Be honest.

### Fidelity Score: X/10

| Score | Meaning |
|-------|---------|
| 9-10 | They would read this and say "I could have written that." Nearly all VAULT-sourced. |
| 7-8 | Captures the right positions and mostly the right voice. Some inference required. |
| 5-6 | Directionally correct but relies on significant inference. Voice is approximate. |
| 3-4 | More Claude than the user. Positions are plausible but not well-evidenced. |
| 1-2 | Fabrication. The vault doesn't support this answer. |

### Honest Assessment

One paragraph. Was this ghost answer worth giving? What would make it better? If the answer is "the vault needs more material on this topic," say so directly.

Mark Phase 5 complete.

---

## Output Format

```
GHOST: [Question]
Question type: [Factual / Opinion / Advice / Personal / Prediction]
Vault coverage: [Strong / Moderate / Thin / Minimal]

---

### The Answer (as the user)

[Ghost answer with inline source tags]

---

### Behind the Curtain

[Evidence map table]

[Source distribution]

[Vault gaps]

[Voice fidelity notes]

**Fidelity score: X/10**

[Honest assessment]
```

---

## Output Guidelines

- The ghost answer should feel like reading something the user wrote, not something written about them.
- Honesty about fidelity is more valuable than a convincing performance. A 4/10 answer that knows it's a 4 is better than a fabricated 8.
- Cite specific notes and dates in the evidence map.
- The Behind the Curtain section is not optional. It's the whole point. Anyone can generate a plausible-sounding answer. The value is in knowing how much of it is real.
- If the vault can't answer the question, say so. That's a valid and valuable output.
- **Screen output only.** Do not create or modify any vault files. The user copies what they want.

## Skipping Logic

| Condition | Behavior |
|-----------|----------|
| No question provided | Ask the user for a question |
| < 3 daily notes and no context files | Report "Insufficient vault data for a ghost answer" |
| Vault has zero relevant content after full search | Report what was searched, what was found (nothing), and what the vault would need |
| >50% EXTRAPOLATED or UNKNOWN | Decline ghost answer; report what the vault contains and where gaps are |

## Integration

Works with:
- `/challenge` -- Challenge pressure-tests the ghost answer's underlying beliefs
- `/trace` -- Trace how the author's position on this topic evolved over time
- `/contradict` -- Find if the ghost answer contradicts other held beliefs
- `/create-context` -- If vault gaps are significant, create context to fill them
- `/drift` -- Compare what the ghost answer says matters vs. actual behavior
- `/close-day` -- Ghost insights can inform daily reflection
