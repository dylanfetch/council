# Council

Assemble councils of AI advisor personas. Consult them on your decisions.

Council is a Claude Code plugin. No server, no dependencies, no API key beyond your Claude subscription. It works by teaching your agent how to coordinate subagent personas through structured deliberation styles.

Think of it as a paradigm, not an application. Personas are markdown files. Deliberation styles are markdown files. Councils are markdown files. Your agent reads them and follows the instructions.

## Install

Council is distributed as a Claude Code plugin marketplace. To install:

1. In Claude Code, run `/plugin` to open the plugin GUI.
2. Choose **Add marketplace** and paste the repo URL: `https://github.com/dylanfetch/council` (or a local clone path during development).
3. From the marketplace list, select **council** and install it.

Once installed, the three skills (`/create-persona`, `/create-council`, `/ask-council`) become available in any Claude Code session.

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


## Tips

The deliberation process is a time-consuming, token-intensive, and context-heavy process. It is recommended that you do your setup in a separate thread from where you actually run your deliberations.

As context windows fill, LLM response quality decreases. We have to be judicious with our use of the orchestrator agent's context window. The less context we use before synthesis, the higher quality the synthesis will be.

The system is designed so that sub-agents do most communication through file writing rather than message exchange with the orchestrator. That's done so that the orchestrator's context remains clean. Subagents read their personas from file. They read and write opinions to file.