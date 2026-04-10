# Council — AI Advisory Council Plugin for Claude Code

**Date:** 2026-04-09
**Status:** Design approved, pending implementation

## Overview

Council is a Claude Code plugin that lets you assemble councils of AI advisor personas and consult them on questions using structured deliberation styles. It's a paradigm — like Karpathy's autoresearch — not a traditional application. The entire system is markdown files teaching an agent how to coordinate subagents.

No server. No dependencies. No API key beyond your existing Claude subscription. Install the plugin, create some personas, form a council, ask a question.

## Core Principles

- **Everything is markdown.** Personas, councils, deliberation results, and skills are all markdown files with YAML frontmatter.
- **The coordinator dispatches, subagents read and write, the filesystem is shared state.** The coordinator (the user's main Claude Code instance) never loads persona content or deliberation text into its own context. It tells subagents which files to read and write. This keeps the coordinator's context small regardless of council size.
- **Deliberation styles are skills, not code.** Anyone can create a new deliberation style by writing a markdown file that describes a coordination pattern. No Python, no server.
- **Personas are the product.** Each persona deserves careful research and craft. They ship separately from the plugin, built one at a time with care.
- **The agent harness is the setup and configuration layer.** Installation, persona creation, council management — all happen through skills that the user invokes via their agent. The deliberation runtime is also the agent, coordinating subagents.

## File Formats

### Persona (`~/.council/personas/<slug>.md`)

```markdown
---
name: Warren Buffett
slug: warren-buffett
role: Investment advisor
era: Contemporary (1930-present)
domain: Value investing, business strategy, capital allocation
style: Folksy, uses metaphors, thinks in decades not quarters
---

# Warren Buffett

## Background
[Rich description of who this person is, their philosophy, key ideas]

## Advisory Style
[How they approach giving advice — what they emphasize, what they dismiss]

## Key Principles
- [Principle 1]
- [Principle 2]

## Signature Phrases
- [Notable quotes that capture their voice]

## Blind Spots
[What this advisor tends to miss or undervalue — important for council diversity]
```

The frontmatter is machine-readable for council assembly, listing, and filtering. The body is the subagent's context — it reads this file to become the persona. The template is a starting point, not a rigid schema. Persona authors can add whatever sections serve the character.

### Council (`~/.council/councils/<slug>.md`)

```markdown
---
name: Investment Brain Trust
members:
  - warren-buffett
  - charlie-munger
  - ray-dalio
default_style: roundtable
---

Advisory council for investment strategy and capital allocation
decisions. Buffett and Munger provide the value investing lens,
Dalio brings the macro/all-weather perspective.
```

A name, a list of persona slugs, a default deliberation style, and an optional prose description. The complexity lives in the personas and the styles, not the config.

### Deliberation Result (`.council/history/<timestamp>_<slug>/result.md`)

```markdown
# Should I allocate to bonds right now?

**Council:** Investment Brain Trust
**Style:** Telephone
**Date:** 2026-04-09

## Step 1 — Warren Buffett
[Buffett's response]

## Step 2 — Charlie Munger
[Munger's response, building on Buffett]

## Step 3 — Ray Dalio
[Dalio's response, building on the full chain]

## Synthesis
[Key agreements, disagreements, and the overall shape of the council's advice]
```

Results are human-readable markdown. They live in `.council/history/` in the current project, or `~/.council/history/` if no project context. Each deliberation gets a timestamped directory.

## Plugin Structure

```
council/
  plugin.json                # plugin manifest
  skills/
    create-persona.md        # research a figure, fill the template
    create-council.md        # assemble personas into a council
    ask-council.md           # entry point: dispatches to a style
    styles/
      telephone.md           # sequential refinement chain
      roundtable.md          # parallel responses → synthesis
  templates/
    persona.md               # blank persona template
```

No bundled personas in v1. The template and the create-persona skill are the starting point. Personas are intensive builds that each deserve their own dedicated effort.

### User data (`~/.council/`)

Created on first use:

```
~/.council/
  personas/                  # user's persona library
  councils/                  # council configurations

.council/                    # project-local (or ~/.council/ if no project)
  history/
    2026-04-09_bonds-allocation/
      step-1-buffett.md
      step-2-munger.md
      step-3-dalio.md
      result.md
```

Personas and councils are global — your library follows you across projects. Deliberation results are project-local when you're in a project, so the council's advice lives next to the work it's about.

## Skills

### `/create-persona`

The user says "create a Marcus Aurelius advisor" or "create an advisor who's a cynical startup veteran."

**Real/historical figure:** The skill instructs the agent to research the figure deeply — philosophy, decision-making patterns, known positions, communication style, documented blind spots. Web search if available. Fills the template, writes to `~/.council/personas/<slug>.md`.

**Original character:** The skill walks through it conversationally — the agent asks a few questions to flesh out the character, then writes the persona. Or the user writes the file themselves from the template.

### `/create-council`

The user says "create a council called 'startup-advisors' with buffett, thiel, and graham."

The skill instructs the agent to:
1. Verify all named personas exist in `~/.council/personas/`
2. If any are missing, offer to create them (invokes create-persona)
3. Ask which default deliberation style to use (or default to roundtable)
4. Write the council config to `~/.council/councils/<slug>.md`

### `/ask-council`

The main event. "Ask my startup-advisors council whether I should raise a seed round."

The skill instructs the agent to:
1. Read the council config to get members and default style
2. Dispatch to the appropriate style skill (telephone, roundtable, etc.)
3. After the style completes, present a synthesis to the user in the terminal
4. Note where the full deliberation is saved

The user can override the style inline: "Ask my startup-advisors (telephone) whether I should raise."

The coordinator's context stays minimal throughout. It reads the council config (a few lines of YAML), follows the style skill instructions, and spawns subagents with file paths. All heavy content — personas, deliberations, synthesis — lives on disk and in subagent contexts.

## Deliberation Styles

### Telephone (sequential refinement)

Each advisor builds on the previous one's response. Good for iteratively sharpening an idea.

```
Session dir: .council/history/<timestamp>_<slug>/

1. Read council config, get ordered member list.
2. For each member in order:
   - If first: spawn subagent with persona path + question.
     Write response to step-1-<slug>.md
   - If subsequent: spawn subagent with persona path + question.
     Read previous step file. Write response (incorporating
     the chain) to step-N-<slug>.md
3. Spawn synthesis subagent: read the final step file,
   produce a summary. Write to result.md
```

The last step file contains the full chain, so the synthesis subagent only reads one file. Each intermediate file is a snapshot of the conversation at that point — you can see how the thinking developed.

### Roundtable (parallel perspectives)

Everyone speaks independently, then everyone reacts. Good for getting diverse perspectives without anchoring bias.

```
Session dir: .council/history/<timestamp>_<slug>/

Round 1 — Independent responses:
1. Spawn ALL members as subagents in parallel.
   Each reads its own persona, writes to round1-<slug>.md
2. Wait for all to complete.

Round 2 — Cross-pollination:
3. Spawn ALL members again in parallel.
   Each reads its own persona + ALL round1-*.md files.
   Writes revised/expanded take to round2-<slug>.md
4. Wait for all to complete.

Synthesis:
5. Spawn synthesis subagent: read all round2-*.md files.
   Identify agreements, disagreements, novel insights.
   Write to result.md
```

Two rounds keeps token usage reasonable. The first round is unbiased. The second round lets members engage with each other.

### Future styles (not in v1)

**Debate:** Adversarial. Two members argue opposing sides, a third judges. Two rounds of argument plus rebuttals, then judgment. Needs at least 3 council members.

**World Cafe:** Themed small-group discussions. A framing subagent breaks the question into dimensions, then each member responds to each theme, then cross-theme synthesis. The heaviest style in tokens, but the most thorough for complex decisions.

### Style authoring

Anyone can write a new style. A style skill needs:
1. A name and description in the frontmatter
2. Clear instructions for the coordinator: what subagents to spawn, in what order, what each one reads and writes
3. A file naming convention for the session directory

No code. Just markdown instructions an agent can follow. The `ask-council` skill discovers available styles by listing the `styles/` directory.

## Persona Sharing

Personas are markdown files. Sharing is just sharing files.

**Plugin repo:** Community-contributed personas go in a `personas/` directory via PR. When the community builds up a critical mass, these can ship with the plugin.

**Persona packs:** A GitHub repo that's a themed collection — "Greek Philosophers Pack," "Silicon Valley Founders Pack." Clone and copy into `~/.council/personas/`.

**Single-file sharing:** Post a persona in a gist, a Discord message, a blog post. Copy to `~/.council/personas/`. It's one file.

No registry, no package manager. The format is the protocol.

## Scope

### v1 includes

- Plugin skeleton and manifest
- Persona template (`templates/persona.md`)
- `/create-persona` skill
- `/create-council` skill
- `/ask-council` skill (dispatcher)
- Telephone style skill
- Roundtable style skill
- File structure conventions (`~/.council/` global, `.council/` project-local)
- README explaining the paradigm, setup, and usage (including VS Code markdown preview: `Ctrl+K` then `V`)

### v1 does not include

- Bundled personas (separate effort, each deserves its own thread)
- Debate and World Cafe styles (built after telephone and roundtable prove the pattern)
- Web viewer / server
- OpenRouter integration
- Token cost estimation
- Council memory across deliberations

### Out of scope

- Database of any kind
- User accounts or auth
- Cloud hosting
- Persona auto-update / versioning
