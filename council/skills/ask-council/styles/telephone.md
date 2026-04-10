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
