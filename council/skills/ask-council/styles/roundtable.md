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
