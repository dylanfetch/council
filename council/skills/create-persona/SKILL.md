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
