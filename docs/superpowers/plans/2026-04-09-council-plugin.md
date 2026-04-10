# Council Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Code plugin that lets users assemble councils of AI advisor personas and consult them via structured deliberation styles — entirely through markdown skills and subagent coordination.

**Architecture:** Everything is a markdown file. Three user-facing skills (`create-persona`, `create-council`, `ask-council`) plus two deliberation style definitions (telephone, roundtable) that `ask-council` dispatches to. No Python, no server, no dependencies. Subagents read persona files and write deliberation results directly — the coordinator never loads heavy content into its own context.

**Tech Stack:** Claude Code plugin system (markdown skills with YAML frontmatter). No runtime dependencies.

**Spec:** `docs/superpowers/specs/2026-04-09-council-plugin-design.md`

**Note on testing:** This project is entirely markdown skill files — there are no automated tests to write. Verification happens in Task 8 by installing the plugin and running each skill against a real Claude Code instance.

---

## File Structure

```
council/                          # new repo at ~/projects/council
  .claude-plugin/
    plugin.json                   # plugin manifest
  package.json                    # minimal npm manifest (required by plugin system)
  skills/
    create-persona/
      SKILL.md                    # research + create advisor persona
    create-council/
      SKILL.md                    # assemble personas into a council
    ask-council/
      SKILL.md                    # dispatcher: reads council, picks style, orchestrates
      styles/
        telephone.md              # sequential refinement coordination pattern
        roundtable.md             # parallel perspectives coordination pattern
  templates/
    persona.md                    # blank persona template
  README.md                       # paradigm explanation, setup, usage
```

---

### Task 1: Plugin scaffold

**Files:**
- Create: `~/projects/council/.claude-plugin/plugin.json`
- Create: `~/projects/council/package.json`

- [ ] **Step 1: Create the project directory and initialize git**

```bash
mkdir -p ~/projects/council/.claude-plugin
mkdir -p ~/projects/council/skills/create-persona
mkdir -p ~/projects/council/skills/create-council
mkdir -p ~/projects/council/skills/ask-council/styles
mkdir -p ~/projects/council/templates
cd ~/projects/council
git init
```

- [ ] **Step 2: Write the plugin manifest**

Create `.claude-plugin/plugin.json`:

```json
{
  "name": "council",
  "description": "Assemble councils of AI advisor personas and consult them using structured deliberation styles",
  "version": "0.1.0",
  "author": {
    "name": "Dylan"
  },
  "license": "MIT",
  "keywords": ["council", "advisors", "personas", "deliberation", "agents"]
}
```

- [ ] **Step 3: Write the package.json**

Create `package.json`:

```json
{
  "name": "council",
  "version": "0.1.0",
  "type": "module"
}
```

- [ ] **Step 4: Create .gitignore**

Create `.gitignore`:

```
node_modules/
.council/
```

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin/plugin.json package.json .gitignore
git commit -m "chore: initialize council plugin scaffold"
```

---

### Task 2: Persona template

**Files:**
- Create: `~/projects/council/templates/persona.md`

- [ ] **Step 1: Write the persona template**

Create `templates/persona.md`:

```markdown
---
name:
slug:
role:
era:
domain:
style:
---

# {name}

## Background

{A rich description of who this person is. For real figures: their history, philosophy,
and the experiences that shaped their worldview. For fictional characters: their
backstory, expertise, and what drives them.}

## Advisory Style

{How this advisor approaches giving advice. What do they emphasize? What do they
dismiss? Do they lead with questions or answers? Are they direct or Socratic?
Optimistic or cautious?}

## Key Principles

- {Core belief or framework that guides their thinking}
- {Another principle}
- {Add as many as characterize this advisor}

## Signature Phrases

- {A quote or turn of phrase this advisor is known for}
- {Another characteristic expression}

## Blind Spots

{What this advisor tends to miss, undervalue, or get wrong. Every great thinker
has limitations. Documenting them makes the council more useful — the user knows
where to watch for gaps.}
```

- [ ] **Step 2: Commit**

```bash
git add templates/persona.md
git commit -m "feat: add persona template"
```

---

### Task 3: create-persona skill

**Files:**
- Create: `~/projects/council/skills/create-persona/SKILL.md`

- [ ] **Step 1: Write the create-persona skill**

Create `skills/create-persona/SKILL.md`:

````markdown
---
name: create-persona
description: Research and create an AI advisor persona for use in councils. Use when the user wants to add a new advisor — real, historical, or fictional.
---

# Create Persona

Create a new advisor persona for the user's council library.

## Setup

Ensure `~/.council/personas/` exists. Create it if it doesn't:

```bash
mkdir -p ~/.council/personas
```

## Determine Persona Type

**Real or historical figure** (e.g., "Warren Buffett", "Marcus Aurelius", "Ada Lovelace"):
- Proceed to the Research section below.

**Fictional or archetype character** (e.g., "a cynical startup veteran", "a cautious CFO"):
- Ask the user 2-3 quick questions to flesh out the character:
  1. What domain or expertise should this advisor have?
  2. What's their personality and communication style?
  3. What perspective do they bring that others might not?
- Then proceed to the Write section below.

## Research (real figures only)

Research the figure thoroughly. Focus on:

1. **Their philosophy and worldview** — what do they believe about their domain? What frameworks do they use to make decisions?
2. **Decision-making patterns** — how do they approach problems? What do they prioritize? What do they ignore?
3. **Communication style** — are they formal or folksy? Do they use metaphors, data, stories, first principles?
4. **Known positions** — what stances are they famous for? What advice have they given publicly?
5. **Documented blind spots** — where have they been wrong? What do critics say they miss?
6. **Signature phrases** — real quotes that capture their voice.

Use web search if available for additional depth. The goal is a persona rich enough that the advisor's responses feel authentic, not generic.

## Write the Persona

Read the persona template from this plugin's `templates/persona.md` for the structure.

Generate the slug from the persona name: lowercase, hyphens for spaces, alphanumeric and hyphens only. Examples: `warren-buffett`, `marcus-aurelius`, `cynical-startup-veteran`.

Fill out every section of the template:
- **Frontmatter:** All fields populated. `style` should be a brief description of communication style.
- **Background:** At least 2-3 paragraphs. This is the richest section — it's what gives the advisor depth.
- **Advisory Style:** How they give advice specifically. Not just personality, but methodology.
- **Key Principles:** 4-6 principles that define their thinking.
- **Signature Phrases:** 3-5 quotes or characteristic expressions.
- **Blind Spots:** Be honest. Every thinker has limitations. This section makes the council more useful.

Add additional sections if they serve the character. The template is a starting point, not a rigid schema.

Write the file to `~/.council/personas/<slug>.md`.

## Show the User

After writing, display the persona's name, role, domain, advisory style summary, and blind spots in the terminal so the user can see what was created without having to open the file.

Tell the user the file path so they can review or edit the full persona.
````

- [ ] **Step 2: Commit**

```bash
git add skills/create-persona/SKILL.md
git commit -m "feat: add create-persona skill"
```

---

### Task 4: create-council skill

**Files:**
- Create: `~/projects/council/skills/create-council/SKILL.md`

- [ ] **Step 1: Write the create-council skill**

````markdown
---
name: create-council
description: Assemble advisor personas into a named council. Use when the user wants to create, list, or manage councils.
---

# Create Council

Assemble existing personas into a named council.

## Setup

Ensure `~/.council/councils/` exists. Create it if it doesn't:

```bash
mkdir -p ~/.council/councils
```

## Parse the Request

Extract from the user's message:
- **Council name** (required)
- **Member names or slugs** (required — at least 2)
- **Default deliberation style** (optional — defaults to `roundtable`)

If the user didn't provide enough information, ask for what's missing.

## Verify Members

For each named member, check if `~/.council/personas/<slug>.md` exists.

To match names to slugs: convert the name to lowercase, replace spaces with hyphens, strip non-alphanumeric characters except hyphens.

If a persona file is missing:
1. Tell the user which personas were not found.
2. List available personas from `~/.council/personas/` so they can check for typos.
3. Offer to create missing personas using the `/create-persona` skill.

Do not proceed until all members are verified.

## Set Default Style

If the user specified a style, use it. Otherwise default to `roundtable`.

Available styles: `telephone`, `roundtable`.

If the user requests a style that doesn't exist, tell them the available options.

## Write the Council File

Generate the council slug from the name: lowercase, hyphens for spaces.

Write to `~/.council/councils/<slug>.md`:

```markdown
---
name: {Council Name}
members:
  - {member-1-slug}
  - {member-2-slug}
  - {member-3-slug}
default_style: {style}
---

{Optional description — include if the user provided context about the council's purpose.
Otherwise leave the body empty.}
```

## Confirm

Show the user:
- Council name and file path
- Member list (names, not just slugs)
- Default deliberation style

Tell them they can now use `/ask-council {slug}` to consult this council.
````

- [ ] **Step 2: Commit**

```bash
git add skills/create-council/SKILL.md
git commit -m "feat: add create-council skill"
```

---

### Task 5: Telephone style definition

**Files:**
- Create: `~/projects/council/skills/ask-council/styles/telephone.md`

- [ ] **Step 1: Write the telephone style**

````markdown
---
name: telephone
description: Sequential refinement — each advisor builds on the previous one's response
---

# Telephone Style

Sequential refinement. Each advisor builds on the last. Good for iteratively sharpening an idea.

## Coordination Instructions

You are the coordinator. You have been given:
- `MEMBERS`: a list of council member persona file paths
- `QUESTION`: the user's question
- `SESSION_DIR`: the directory for this deliberation's files

**IMPORTANT: Do NOT read persona files yourself.** Subagents read their own personas. This keeps your context clean.

## Procedure

### Step 1 — First member

Spawn a subagent with this prompt (fill in the bracketed values):

```
You are an AI advisor embodying a specific persona.

1. Read your persona file: [FIRST_MEMBER_PERSONA_PATH]
2. Fully embody this advisor's perspective, voice, and thinking style.
3. Consider the following question:

"[QUESTION]"

4. Write your response to: [SESSION_DIR]/step-1-[MEMBER_SLUG].md

Format your file like this:

# [QUESTION]

**Council:** [COUNCIL_NAME]
**Style:** Telephone
**Date:** [TODAY'S DATE]

## Step 1 — [PERSONA_NAME]

[Your response as this advisor. Draw on their known principles, frameworks,
and communication style. Stay in character throughout. Be substantive —
give real advice, not platitudes.]
```

Wait for the subagent to complete before continuing.

### Step 2 through N — Subsequent members

For each remaining member, spawn a subagent with this prompt:

```
You are an AI advisor embodying a specific persona.

1. Read your persona file: [MEMBER_PERSONA_PATH]
2. Fully embody this advisor's perspective, voice, and thinking style.
3. Read the deliberation so far from: [SESSION_DIR]/step-[N-1]-[PREV_MEMBER_SLUG].md
4. Consider the following question:

"[QUESTION]"

5. Write your response to: [SESSION_DIR]/step-[N]-[MEMBER_SLUG].md

Copy the ENTIRE contents of the previous step file into your new file,
then append your own section at the end:

## Step [N] — [PERSONA_NAME]

[Your response. Build on, challenge, or refine what came before. Reference
specific points from prior advisors by name. Stay in character. Be substantive.]
```

Wait for each subagent to complete before spawning the next. The chain is sequential — each advisor must see all prior responses.

### Synthesis

After all members have responded, spawn a final subagent:

```
Read the full deliberation chain from: [SESSION_DIR]/step-[LAST_N]-[LAST_MEMBER_SLUG].md

Append a synthesis section to the end of this file and write the result
to: [SESSION_DIR]/result.md

The synthesis section should be:

## Synthesis

[Summarize the deliberation:
- Where did the advisors agree?
- Where did they disagree, and why?
- What was the most unexpected or valuable insight?
- What is the overall shape of the council's advice?

Be specific. Reference advisors by name. Don't just list — synthesize.]
```

## File Naming

- `step-1-<slug>.md` — first member's response (includes metadata header)
- `step-2-<slug>.md` — full chain through second member
- `step-N-<slug>.md` — full chain through member N
- `result.md` — complete chain plus synthesis (the final deliverable)
````

- [ ] **Step 2: Commit**

```bash
git add skills/ask-council/styles/telephone.md
git commit -m "feat: add telephone deliberation style"
```

---

### Task 6: Roundtable style definition

**Files:**
- Create: `~/projects/council/skills/ask-council/styles/roundtable.md`

- [ ] **Step 1: Write the roundtable style**

````markdown
---
name: roundtable
description: Parallel perspectives — everyone speaks independently, then reacts to each other
---

# Roundtable Style

Everyone speaks, then everyone reacts. Good for diverse perspectives without anchoring bias.

## Coordination Instructions

You are the coordinator. You have been given:
- `MEMBERS`: a list of council member persona file paths
- `QUESTION`: the user's question
- `SESSION_DIR`: the directory for this deliberation's files

**IMPORTANT: Do NOT read persona files yourself.** Subagents read their own personas.

## Procedure

### Round 1 — Independent responses

Spawn ALL members as subagents **in parallel** (use multiple Agent tool calls in a single message). Each subagent gets this prompt:

```
You are an AI advisor embodying a specific persona.

1. Read your persona file: [MEMBER_PERSONA_PATH]
2. Fully embody this advisor's perspective, voice, and thinking style.
3. Consider the following question independently — do not try to guess
   what others might say:

"[QUESTION]"

4. Write your response to: [SESSION_DIR]/round1-[MEMBER_SLUG].md

Format your file like this:

## [PERSONA_NAME] — Initial Response

[Your independent perspective on the question. Draw on your persona's
principles, experience, and frameworks. Be substantive and specific.
Stay in character throughout.]
```

Wait for ALL subagents to complete before moving to Round 2.

### Round 2 — Cross-pollination

Spawn ALL members as subagents **in parallel** again. Each gets this prompt:

```
You are an AI advisor embodying a specific persona.

1. Read your persona file: [MEMBER_PERSONA_PATH]
2. Read ALL Round 1 responses:
   [List each round1-*.md file path explicitly]
3. Reconsider the question in light of what your fellow advisors said:

"[QUESTION]"

4. Write your revised response to: [SESSION_DIR]/round2-[MEMBER_SLUG].md

Format your file like this:

## [PERSONA_NAME] — Revised Response

[Your updated perspective after hearing from the other advisors. You may:
- Strengthen your original position with new arguments
- Update your view based on others' insights
- Call out specific disagreements by name
- Add considerations you missed in Round 1

Reference other advisors by name when engaging with their ideas.
Stay in character throughout.]
```

Wait for ALL subagents to complete before moving to Synthesis.

### Synthesis

Spawn a synthesis subagent:

```
Read all Round 1 and Round 2 responses:
- Round 1: [List each round1-*.md file path explicitly]
- Round 2: [List each round2-*.md file path explicitly]

Write a complete deliberation record to: [SESSION_DIR]/result.md

Format the file like this:

# [QUESTION]

**Council:** [COUNCIL_NAME]
**Style:** Roundtable
**Date:** [TODAY'S DATE]

[For each member, include both their Round 1 and Round 2 responses,
grouped by member:]

## [PERSONA_NAME]

### Initial Response
[Their round1 content]

### Revised Response
[Their round2 content]

[Repeat for each member]

## Synthesis

[Summarize the deliberation:
- Areas of agreement across advisors
- Areas of disagreement and the reasoning on each side
- How perspectives evolved between Round 1 and Round 2
- The most unexpected or valuable insight
- Key questions that remain open

Be specific. Reference advisors by name. Synthesize, don't just list.]
```

## File Naming

- `round1-<slug>.md` — each member's independent response
- `round2-<slug>.md` — each member's revised response
- `result.md` — all responses organized by member, plus synthesis
````

- [ ] **Step 2: Commit**

```bash
git add skills/ask-council/styles/roundtable.md
git commit -m "feat: add roundtable deliberation style"
```

---

### Task 7: ask-council skill

**Files:**
- Create: `~/projects/council/skills/ask-council/SKILL.md`

- [ ] **Step 1: Write the ask-council skill**

````markdown
---
name: ask-council
description: Ask a council of AI advisors a question using a structured deliberation style. Use when the user wants to consult their council on a decision or question.
---

# Ask Council

Consult a council of AI advisor personas on a question.

## Parse the Request

Extract from the user's message:
- **Council name or slug** — which council to consult
- **Style override** (optional) — e.g., "(telephone)" or "using telephone style"
- **Question** — what they want the council's advice on

Examples of valid invocations:
- `/ask-council startup-advisors Should I raise a seed round?`
- `/ask-council startup-advisors (telephone) Should I raise a seed round?`
- `/ask-council` (will prompt for missing info)

If the council name is missing, list available councils from `~/.council/councils/` and ask which one.

If the question is missing, ask for it.

## Read the Council Config

Read `~/.council/councils/<slug>.md`. Extract from YAML frontmatter:
- `name` — council display name
- `members` — list of persona slugs
- `default_style` — fallback deliberation style

Build the persona file paths: `~/.council/personas/<member-slug>.md` for each member.

Verify all persona files exist. If any are missing, tell the user and stop.

## Determine the Style

Use the user's style override if provided. Otherwise use the council's `default_style`.

Available styles and their definition files (relative to this skill's directory):
- `telephone` → `styles/telephone.md`
- `roundtable` → `styles/roundtable.md`

If the requested style doesn't exist, tell the user the available options.

## Create the Session Directory

Generate a question slug from the first 4-6 meaningful words of the question: lowercase, hyphens for spaces, strip non-alphanumeric characters except hyphens.

Determine the session directory:
- If the current working directory appears to be a project (has a `.git` directory, or a `CLAUDE.md`, or similar project markers): use `.council/history/<YYYY-MM-DD>_<question-slug>/`
- Otherwise: use `~/.council/history/<YYYY-MM-DD>_<question-slug>/`

Create the session directory:

```bash
mkdir -p <session-dir>
```

## Announce

Tell the user:
> Running **[style]** deliberation with **[council name]**: [member names joined by ", "]...

## Dispatch to Style

Read the style definition file from this skill's `styles/` directory.

Follow the style's Coordination Instructions, passing:
- `MEMBERS`: the list of persona file paths
- `QUESTION`: the user's question
- `SESSION_DIR`: the session directory path
- `COUNCIL_NAME`: the council's display name

Execute the style's procedure exactly as written.

## Present Results

After the style completes, read `<session-dir>/result.md`.

Present the **Synthesis** section to the user in the terminal. Keep it concise — the full deliberation is in the file.

Tell the user:
> Full deliberation saved to `<session-dir>/result.md`
>
> To view the formatted result in VS Code: open the file and press `Ctrl+K` then `V`.
````

- [ ] **Step 2: Commit**

```bash
git add skills/ask-council/SKILL.md
git commit -m "feat: add ask-council skill"
```

---

### Task 8: README

**Files:**
- Create: `~/projects/council/README.md`

- [ ] **Step 1: Write the README**

Create `README.md`:

````markdown
# Council

Assemble councils of AI advisor personas. Consult them on your decisions.

Council is a Claude Code plugin. No server, no dependencies, no API key beyond your Claude subscription. It works by teaching your agent how to coordinate subagent personas through structured deliberation styles.

Think of it as a paradigm, not an application. Personas are markdown files. Deliberation styles are markdown files. Councils are markdown files. Your agent reads them and follows the instructions.

## Install

```
/plugin install council
```

## Quick Start

**1. Create some advisors:**

```
/create-persona Warren Buffett
/create-persona Charlie Munger
/create-persona Ray Dalio
```

Each persona is a markdown file in `~/.council/personas/`. The skill researches the figure and fills out a detailed persona template covering their background, advisory style, key principles, signature phrases, and blind spots.

**2. Form a council:**

```
/create-council investment-brain-trust with warren-buffett, charlie-munger, ray-dalio
```

**3. Ask a question:**

```
/ask-council investment-brain-trust Should I allocate to bonds right now?
```

Your council deliberates using the default style (roundtable). Each advisor reads their own persona file, responds in character, then reacts to the other advisors. A synthesis at the end captures agreements, disagreements, and key insights.

The full deliberation is saved as markdown in `.council/history/` in your current project.

## Deliberation Styles

**Roundtable** (default): Everyone responds independently, then everyone reads each other's responses and revises. Two rounds. Good for getting diverse perspectives without anchoring bias.

**Telephone**: Sequential refinement. Each advisor builds on the previous one's response. The chain grows as it passes through each member. Good for iteratively sharpening an idea.

Override the default style:

```
/ask-council investment-brain-trust (telephone) Should I allocate to bonds right now?
```

## Personas

A persona is a markdown file with YAML frontmatter:

```yaml
---
name: Warren Buffett
slug: warren-buffett
role: Investment advisor
era: Contemporary (1930-present)
domain: Value investing, business strategy
style: Folksy, uses metaphors, thinks in decades
---
```

The body contains sections for Background, Advisory Style, Key Principles, Signature Phrases, and Blind Spots. See `templates/persona.md` for the full template.

**Create your own:** Copy the template to `~/.council/personas/<slug>.md` and fill it out. Or use `/create-persona` and let the agent do the research.

**Share personas:** They're just files. Share them however you share files — gists, repos, messages. Drop them in `~/.council/personas/` and they're ready to use.

## Councils

A council is a markdown file listing which personas to include:

```yaml
---
name: Investment Brain Trust
members:
  - warren-buffett
  - charlie-munger
  - ray-dalio
default_style: roundtable
---
```

Councils live in `~/.council/councils/`. Deliberation results live in `.council/history/` in your current project (or `~/.council/history/` when not in a project).

## Viewing Results

Deliberation results are markdown files. Open them in your editor.

**VS Code:** Open the result file and press `Ctrl+K` then `V` to see the formatted markdown side-by-side.

## Writing New Styles

Deliberation styles are markdown files that tell the coordinator how to orchestrate subagents. Look at the existing styles in `skills/ask-council/styles/` for examples.

A style file describes:
1. What subagents to spawn and in what order
2. What each subagent reads (persona files, prior responses)
3. What each subagent writes (step files, round files)
4. How to synthesize the results

No code required. Just instructions an agent can follow.
````

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README"
```

---

### Task 9: Integration test

This task verifies the plugin works end-to-end. It requires a real Claude Code instance.

- [ ] **Step 1: Install the plugin**

From the council project directory:

```
/plugin install ~/projects/council
```

Or if Claude Code requires a different installation path for local plugins, use:

```
/plugin install file:///home/dylan/projects/council
```

Verify the plugin appears in the installed plugins list and all three skills are discoverable.

- [ ] **Step 2: Test create-persona**

```
/create-persona Marcus Aurelius
```

Verify:
- `~/.council/personas/` directory was created
- `~/.council/personas/marcus-aurelius.md` exists
- File has valid YAML frontmatter with all fields populated
- All body sections (Background, Advisory Style, Key Principles, Signature Phrases, Blind Spots) are filled out
- The persona feels authentic, not generic

- [ ] **Step 3: Create a second persona for council testing**

```
/create-persona Seneca
```

Verify `~/.council/personas/seneca.md` exists and is well-formed.

- [ ] **Step 4: Test create-council**

```
/create-council stoic-advisors with marcus-aurelius, seneca
```

Verify:
- `~/.council/councils/` directory was created
- `~/.council/councils/stoic-advisors.md` exists
- YAML frontmatter lists both members
- `default_style` is set to `roundtable`

- [ ] **Step 5: Test ask-council with roundtable (default)**

```
/ask-council stoic-advisors How should I deal with a business setback?
```

Verify:
- A session directory was created in `.council/history/` (project-local) or `~/.council/history/`
- `round1-marcus-aurelius.md` and `round1-seneca.md` exist (Round 1)
- `round2-marcus-aurelius.md` and `round2-seneca.md` exist (Round 2)
- `result.md` exists with the full deliberation and synthesis
- Each advisor stayed in character
- The synthesis references advisors by name and captures agreements/disagreements
- The coordinator's terminal output shows the synthesis and the file path

- [ ] **Step 6: Test ask-council with telephone style override**

```
/ask-council stoic-advisors (telephone) What virtue matters most for leadership?
```

Verify:
- A new session directory was created
- `step-1-marcus-aurelius.md` exists with just the first response
- `step-2-seneca.md` exists with the full chain (Marcus Aurelius + Seneca)
- `result.md` exists with the full chain plus synthesis
- The chain builds correctly — step 2 contains step 1's content plus Seneca's addition

- [ ] **Step 7: Verify context efficiency**

During both ask-council tests, observe the coordinator's behavior:
- The coordinator should NOT have read any persona files directly
- The coordinator should NOT have loaded deliberation file contents into its own context
- The coordinator should only have read the council config and dispatched to subagents

If the coordinator is loading heavy content, the style instructions need tightening.
