# Council

Assemble councils of AI advisor personas. Consult them on your decisions.

Council is a Claude Code plugin. No server, no dependencies, no API key beyond your Claude subscription. It works by teaching your agent how to coordinate subagent personas through structured deliberation styles.

Think of it as a paradigm, not an application. Personas are markdown files. Deliberation styles are markdown files. Councils are markdown files. Your agent reads them and follows the instructions.

## Flexible and Extensible

This plugin is built to be customized:

- create your own pool of advisors
  - real advisors get researched by Claude
  - fictional advisors are specified by you
- create your own set of councils
  - use different combinations of advisors to address different prompts
- create your own deliberation styles
  - instruct your advisors how to interact

## Install

Council is distributed as a Claude Code plugin marketplace. To install:

1. In the Claude Code VS Code Extension, run `/plugins` to open the plugin GUI.
2. Choose **Add marketplace** and paste the repo URL: `https://github.com/dylanfetch/council`.
3. From the marketplace list, select **council** and install it.

Once installed, the three skills (`/create-persona`, `/create-council`, `/ask-council`) become available in any Claude Code session.

## Quick Start

**1. Create some advisors:**

```
/create-persona Warren Buffett
/create-persona Tim Ferriss
/create-persona Cal Newport
/create-persona James Clear
/create-persona Marcus Aurelius
```

Each persona is a markdown file in `~/.council/personas/`. The skill researches the figure and fills out a detailed persona template covering their background, advisory style, key principles, signature phrases, and blind spots.

**2. Form a council:**

```
/create-council personal-advisors with warren-buffett, tim-ferriss, cal-newport, james-clear, marcus-aurelius
```

**3. Ask a question:**

```
/ask-council personal-advisors What should a 30-year-old optimize for now so that, at 80, they are glad they lived the way they did?
```

Your council deliberates using the default style (roundtable). Each advisor reads their own persona file, responds in character, then reacts to the other advisors. A synthesis at the end captures agreements, disagreements, and key insights.

The full deliberation is saved as markdown in `.council/history/` in your current project.

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

## Deliberation Styles

Two styles ship with the plugin:

- **Roundtable** (default): Everyone responds independently, then everyone reads each other's responses and revises. Two rounds. Good for getting diverse perspectives without anchoring bias.
- **Telephone**: Sequential refinement. Each advisor builds on the previous one's response. The chain grows as it passes through each member. Good for iteratively sharpening an idea.

Override the default style when asking a question:

```
/ask-council personal-advisors (telephone) Choose one principle, habit, or constraint a person should adopt to build a life that is wealthy, focused, disciplined, and meaningful over the long term
```

**Create your own:** Copy an existing style from `skills/ask-council/styles/` and adapt it to your coordination pattern.

A deliberation style is a markdown file with YAML frontmatter that tells the coordinator how to orchestrate subagents:

```yaml
---
name: roundtable
description: Parallel perspectives — everyone speaks independently, then reacts to each other
---
```

The body describes what subagents to spawn and in what order, what each subagent reads (persona files, prior responses), what each subagent writes (step files, round files), and how to synthesize the results. No code required — just instructions an agent can follow.


## Viewing Results

Deliberation results are markdown files. Open them in your editor.

**VS Code:** Open the result file and press `Ctrl+K` then `V` to see the formatted markdown side-by-side.

## Tips

The deliberation process is a time-consuming, token-intensive, and context-heavy process. It is recommended that you do your setup in a separate thread from where you actually run your deliberations.

As context windows fill, LLM response quality decreases. We have to be judicious with our use of the orchestrator agent's context window. The less context we use before synthesis, the higher quality the synthesis will be.

The system is designed so that sub-agents do most communication through file writing rather than message exchange with the orchestrator. That's done so that the orchestrator's context remains clean. Subagents read their personas from file. They read and write opinions to file.