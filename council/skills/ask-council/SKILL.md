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
