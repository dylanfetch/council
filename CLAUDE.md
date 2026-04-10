# CLAUDE.md

## Project Overview

Council is a Claude Code plugin — a paradigm for assembling councils of AI advisor personas and consulting them using structured deliberation styles. Everything is markdown files teaching an agent how to coordinate subagents. No server, no dependencies, no code.

See [design spec](docs/superpowers/specs/2026-04-09-council-plugin-design.md) for the full rationale and [implementation plan](docs/superpowers/plans/2026-04-09-council-plugin.md) for build details.

## Architecture

This is a Claude Code plugin, not a traditional software project. There is no code to run, no tests to execute, no build step. The "product" is a set of markdown skill files that instruct an AI agent.

### Repository structure

The repo is a single-plugin marketplace. The marketplace.json lives at the repo root and points at `./council/`, which is the actual plugin:

```
.claude-plugin/
  marketplace.json               # local marketplace registration (enables /plugin install)
council/                         # the plugin root (source in marketplace.json)
  .claude-plugin/
    plugin.json                  # plugin manifest (name, version, description)
  skills/
    create-persona/SKILL.md      # skill: research + create advisor persona
    create-council/SKILL.md      # skill: assemble personas into a council
    ask-council/
      SKILL.md                   # skill: dispatcher — reads council, picks style, orchestrates
      styles/
        telephone.md             # style: sequential refinement coordination pattern
        roundtable.md            # style: parallel perspectives coordination pattern
  templates/
    persona.md                   # blank persona template with guidance text
CLAUDE.md                        # dev docs (this file — stays at root)
README.md                        # user-facing readme
docs/                            # design specs and implementation plans
```

The plugin must live in a subdirectory (not the repo root) because the marketplace schema requires `plugins[].source` to be a relative path to a subdirectory — `"."` and `"./"` are both rejected.

### User data (created at runtime, not in this repo)

```
~/.council/
  personas/<slug>.md             # global persona library
  councils/<slug>.md             # council configurations
  history/                       # fallback location for deliberation results

.council/                        # project-local deliberation results
  history/<YYYY-MM-DD>_<slug>/
    step-N-<slug>.md             # telephone step files
    round1-<slug>.md             # roundtable round files
    round2-<slug>.md
    result.md                    # final deliverable with synthesis
```

### How skills work

Skills are markdown files with YAML frontmatter (`name`, `description`). Each lives in `skills/<name>/SKILL.md`. The plugin system discovers them automatically.

**Style files are NOT skills.** They're supporting files inside `skills/ask-council/styles/`. The `ask-council` skill reads them and follows their coordination instructions. Users don't invoke styles directly.

### Deliberation flow

1. User invokes `/ask-council` with a council name and question
2. `ask-council` reads the council config, determines the style, creates a session directory
3. `ask-council` reads the appropriate style file and follows its instructions
4. The style spawns subagents — each reads its own persona file, writes its response to disk
5. A synthesis subagent reads the final outputs and writes `result.md`
6. `ask-council` presents the synthesis to the user

**Critical principle:** The coordinator (main Claude Code instance running ask-council) never reads persona files or deliberation content into its own context. It only dispatches subagents with file paths. All heavy content stays in subagent contexts and on disk.

## Design principles to preserve

- **Everything is markdown.** Personas, councils, results, styles, and skills are all markdown with YAML frontmatter. No JSON, no databases, no code.
- **Coordinator dispatches, subagents read and write.** The filesystem is the shared state. The coordinator's context stays small regardless of council size.
- **Personas are the product.** Each persona deserves thorough research and careful craft. Don't generate them casually or in bulk.
- **Styles are composable.** Anyone can write a new deliberation style by dropping a markdown file in `styles/`. A style just describes a coordination pattern an agent can follow.
- **Dead simple for the user.** Install plugin, create personas, form council, ask question. Four steps to value.

## How to develop

### Adding a new deliberation style

Create `council/skills/ask-council/styles/<name>.md` with:
- YAML frontmatter: `name` and `description`
- Coordination Instructions section that receives `MEMBERS`, `QUESTION`, `SESSION_DIR`, `COUNCIL_NAME`
- A procedure the coordinator follows to spawn and coordinate subagents
- A file naming section documenting the convention
- Must include "Do NOT read persona files yourself" instruction

Look at `telephone.md` and `roundtable.md` as reference. Update the available styles list in `council/skills/ask-council/SKILL.md` and `council/skills/create-council/SKILL.md`.

### Adding a new skill

Create `council/skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`). The description should start with "Use when..." to help Claude Code match user intent to the skill.

### Editing existing skills

Read the skill file before modifying. Skill instructions must be precise enough that an agent can follow them without ambiguity. When editing subagent prompts in style files, be especially careful — those prompts are the runtime.

### Creating personas

Personas are built in separate dedicated sessions, not as part of plugin development. Each persona deserves its own focused effort with thorough research. Use `/create-persona` to build them.

## Testing

There are no automated tests. The plugin is tested by installing it in Claude Code and invoking each skill. See Task 9 in the [implementation plan](docs/superpowers/plans/2026-04-09-council-plugin.md) for the full test procedure.

### Installing locally for testing

`/plugin install` does NOT accept a filesystem path — it only installs named plugins from an already-added marketplace. There are two ways to load this plugin locally:

**Option A — `--plugin-dir` flag (fastest for dev iteration):**
```
claude --plugin-dir ~/projects/council
```
Loads the plugin directly at launch. No marketplace, no install step.

**Option B — local marketplace (exercises the real install path):**
```
/plugin marketplace add ~/projects/council
/plugin install council@council
```
This repo already contains `.claude-plugin/marketplace.json` declaring a single-plugin marketplace named `council`, so the commands above work as-is.
