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
