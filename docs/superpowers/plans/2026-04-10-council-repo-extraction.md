# Council Repo Extraction Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extract the Council Claude Code plugin from the experimental worktree into a fresh standalone public GitHub repository at `dylanfetch/council` (or `dylanfetch/agent-council` fallback).

**Architecture:** Fresh `git init` at `/home/dylan/projects/<repo-name>`, copy files from the source worktree, fix several in-flight updates (marketplace name, gitignore, CLAUDE.md paths, README install section), write MIT LICENSE, single initial commit, push to the user-created empty GitHub repo.

**Tech Stack:** bash, git, filesystem (no build system, no test framework — this is a file-move + git operation).

**Source worktree:** `/home/dylan/projects/agent-conference-experimental` (experimental branch of agent-conference repo). **This source must remain untouched after extraction.**

**Spec:** [docs/superpowers/specs/2026-04-10-council-repo-extraction-design.md](../specs/2026-04-10-council-repo-extraction-design.md)

---

## Parameters

These values are fixed for this plan. If the fallback is needed, replace `council` with `agent-council` in every path and command.

| Parameter | Value |
|---|---|
| `SRC` | `/home/dylan/projects/agent-conference-experimental` |
| `DEST` | `/home/dylan/projects/council` (fallback: `/home/dylan/projects/agent-council`) |
| `REPO_NAME` | `council` (fallback: `agent-council`) |
| `MARKETPLACE_NAME` | `council` (was `council-local`) |
| `LICENSE_HOLDER` | `Dylan Fetch` |
| `LICENSE_YEAR` | `2026` |
| `GITHUB_URL` | `https://github.com/dylanfetch/council.git` |

---

## Task 1: Create destination directory and initialize git

**Files:**
- Create: `/home/dylan/projects/council/` (directory)
- Create: `/home/dylan/projects/council/.git/` (via `git init`)

- [ ] **Step 1: Verify destination does not already exist**

Run: `test ! -e /home/dylan/projects/council && echo "clear" || echo "EXISTS — STOP"`
Expected: `clear`

If output is `EXISTS — STOP`: try the fallback name `agent-council` and use that throughout the rest of the plan. If the fallback also exists, halt and ask the user.

- [ ] **Step 2: Capture the source worktree baseline**

Run: `git -C /home/dylan/projects/agent-conference-experimental status --short && git -C /home/dylan/projects/agent-conference-experimental log -1 --format='%H %s'`
Expected: empty first line (no uncommitted changes), second line is the current HEAD SHA and commit subject.

Record this HEAD SHA — Task 11 will compare against it to verify the source worktree was not touched during extraction.

- [ ] **Step 3: Create the destination directory**

Run: `mkdir /home/dylan/projects/council`
Expected: no output, exit 0.

- [ ] **Step 4: Initialize git with main as the default branch**

Run: `git -C /home/dylan/projects/council init -b main`
Expected: output containing `Initialized empty Git repository in /home/dylan/projects/council/.git/`

- [ ] **Step 5: Verify the init**

Run: `git -C /home/dylan/projects/council status && git -C /home/dylan/projects/council branch --show-current`
Expected: `On branch main`, `No commits yet`, `nothing to commit`, and `main` as current branch.

---

## Task 2: Copy the plugin directory and verbatim files

**Files:**
- Copy: `council/` (entire tree)
- Copy: `IDEAS.md`
- Copy: `.gitignore` (will be edited in Task 3)
- Copy: `.claude-plugin/marketplace.json` (will be edited in Task 4)
- Copy: `README.md` (will be edited in Task 6)
- Copy: `CLAUDE.md` (will be edited in Task 5)
- Copy: `docs/superpowers/specs/2026-04-09-council-plugin-design.md`
- Copy: `docs/superpowers/specs/2026-04-10-council-repo-extraction-design.md`
- Copy: `docs/superpowers/plans/2026-04-09-council-plugin.md`
- Copy: `docs/superpowers/plans/2026-04-10-council-repo-extraction.md` (this file)

- [ ] **Step 1: Copy the plugin directory recursively**

Run: `cp -r /home/dylan/projects/agent-conference-experimental/council /home/dylan/projects/council/council`
Expected: no output.

- [ ] **Step 2: Verify the plugin tree landed intact**

Run: `find /home/dylan/projects/council/council -type f | sort`
Expected exactly these files:
```
/home/dylan/projects/council/council/.claude-plugin/plugin.json
/home/dylan/projects/council/council/skills/ask-council/SKILL.md
/home/dylan/projects/council/council/skills/ask-council/styles/roundtable.md
/home/dylan/projects/council/council/skills/ask-council/styles/telephone.md
/home/dylan/projects/council/council/skills/create-council/SKILL.md
/home/dylan/projects/council/council/skills/create-persona/SKILL.md
/home/dylan/projects/council/council/templates/persona.md
```

- [ ] **Step 3: Copy the marketplace directory**

Run: `cp -r /home/dylan/projects/agent-conference-experimental/.claude-plugin /home/dylan/projects/council/.claude-plugin`
Expected: no output.

- [ ] **Step 4: Copy root-level files**

Run:
```bash
cp /home/dylan/projects/agent-conference-experimental/IDEAS.md /home/dylan/projects/council/IDEAS.md
cp /home/dylan/projects/agent-conference-experimental/.gitignore /home/dylan/projects/council/.gitignore
cp /home/dylan/projects/agent-conference-experimental/README.md /home/dylan/projects/council/README.md
cp /home/dylan/projects/agent-conference-experimental/CLAUDE.md /home/dylan/projects/council/CLAUDE.md
```
Expected: no output.

- [ ] **Step 5: Create the docs subdirectories and copy the four design/plan files**

Run:
```bash
mkdir -p /home/dylan/projects/council/docs/superpowers/specs /home/dylan/projects/council/docs/superpowers/plans
cp /home/dylan/projects/agent-conference-experimental/docs/superpowers/specs/2026-04-09-council-plugin-design.md /home/dylan/projects/council/docs/superpowers/specs/
cp /home/dylan/projects/agent-conference-experimental/docs/superpowers/specs/2026-04-10-council-repo-extraction-design.md /home/dylan/projects/council/docs/superpowers/specs/
cp /home/dylan/projects/agent-conference-experimental/docs/superpowers/plans/2026-04-09-council-plugin.md /home/dylan/projects/council/docs/superpowers/plans/
cp /home/dylan/projects/agent-conference-experimental/docs/superpowers/plans/2026-04-10-council-repo-extraction.md /home/dylan/projects/council/docs/superpowers/plans/
```
Expected: no output.

- [ ] **Step 6: Verify the full destination tree matches expectations**

Run: `find /home/dylan/projects/council -type f -not -path '*/.git/*' | sort`
Expected exactly these 16 files:
```
/home/dylan/projects/council/.claude-plugin/marketplace.json
/home/dylan/projects/council/.gitignore
/home/dylan/projects/council/CLAUDE.md
/home/dylan/projects/council/IDEAS.md
/home/dylan/projects/council/README.md
/home/dylan/projects/council/council/.claude-plugin/plugin.json
/home/dylan/projects/council/council/skills/ask-council/SKILL.md
/home/dylan/projects/council/council/skills/ask-council/styles/roundtable.md
/home/dylan/projects/council/council/skills/ask-council/styles/telephone.md
/home/dylan/projects/council/council/skills/create-council/SKILL.md
/home/dylan/projects/council/council/skills/create-persona/SKILL.md
/home/dylan/projects/council/council/templates/persona.md
/home/dylan/projects/council/docs/superpowers/plans/2026-04-09-council-plugin.md
/home/dylan/projects/council/docs/superpowers/plans/2026-04-10-council-repo-extraction.md
/home/dylan/projects/council/docs/superpowers/specs/2026-04-09-council-plugin-design.md
/home/dylan/projects/council/docs/superpowers/specs/2026-04-10-council-repo-extraction-design.md
```

Note: `LICENSE` is not in this list yet — it's created in Task 7, bringing the total to 17.

---

## Task 3: Fix `.gitignore` — remove the `docs/` line

**Files:**
- Modify: `/home/dylan/projects/council/.gitignore`

**Before:**
```
node_modules/
.council/
.superpowers/
__pycache__/
*.egg-info/
.env
data/
docs/
```

**After:**
```
node_modules/
.council/
.superpowers/
__pycache__/
*.egg-info/
.env
data/
```

- [ ] **Step 1: Delete the `docs/` line**

Use the Edit tool:
- `file_path`: `/home/dylan/projects/council/.gitignore`
- `old_string`: `data/\ndocs/\n`
- `new_string`: `data/\n`

- [ ] **Step 2: Verify the new content**

Read `/home/dylan/projects/council/.gitignore`. Confirm: 7 lines total, no `docs/` line, ends with `data/` followed by newline.

---

## Task 4: Update `marketplace.json` — rename marketplace

**Files:**
- Modify: `/home/dylan/projects/council/.claude-plugin/marketplace.json`

- [ ] **Step 1: Change the top-level `name` from `council-local` to `council`**

Use the Edit tool:
- `file_path`: `/home/dylan/projects/council/.claude-plugin/marketplace.json`
- `old_string`: `"name": "council-local",`
- `new_string`: `"name": "council",`

- [ ] **Step 2: Verify**

Read `/home/dylan/projects/council/.claude-plugin/marketplace.json`. Confirm:
- Top-level `name` is `"council"`
- `owner.name` is `"Dylan"` (unchanged)
- `plugins[0].name` is `"council"` (unchanged)
- `plugins[0].source` is `"./council"` (unchanged)
- `plugins[0].description` is unchanged

---

## Task 5: Update `CLAUDE.md` — paths and marketplace references

**Files:**
- Modify: `/home/dylan/projects/council/CLAUDE.md`

The source `CLAUDE.md` has these stale strings that must be replaced throughout:
- `~/projects/agent-conference-experimental` → `~/projects/council`
- `agent-conference-experimental` → `council` (remaining bare occurrences after the above)
- `dylan-local` → `council`

- [ ] **Step 1: Read the file to find all occurrences**

Read `/home/dylan/projects/council/CLAUDE.md` in full. Note every line number where any of the three stale strings appear.

- [ ] **Step 2: Replace `~/projects/agent-conference-experimental` → `~/projects/council`**

Use the Edit tool with `replace_all: true`:
- `file_path`: `/home/dylan/projects/council/CLAUDE.md`
- `old_string`: `~/projects/agent-conference-experimental`
- `new_string`: `~/projects/council`

- [ ] **Step 3: Replace any remaining bare `agent-conference-experimental` → `council`**

Grep the file first: `Grep` tool on `/home/dylan/projects/council/CLAUDE.md` for pattern `agent-conference-experimental`. If any matches remain after Step 2, use Edit with `replace_all: true`:
- `old_string`: `agent-conference-experimental`
- `new_string`: `council`

If Grep returns no matches, skip this step.

- [ ] **Step 4: Replace `dylan-local` → `council`**

Use the Edit tool with `replace_all: true`:
- `file_path`: `/home/dylan/projects/council/CLAUDE.md`
- `old_string`: `dylan-local`
- `new_string`: `council`

- [ ] **Step 5: Verify no stale references remain**

Run three Grep checks against `/home/dylan/projects/council/CLAUDE.md`:
- Pattern: `agent-conference-experimental` — expected: zero matches
- Pattern: `dylan-local` — expected: zero matches
- Pattern: `council-local` — expected: zero matches

If any check fails, fix manually and re-verify.

- [ ] **Step 6: Sanity read**

Read the "Installing locally for testing" section of `/home/dylan/projects/council/CLAUDE.md`. Confirm the two documented install paths (`--plugin-dir` flag and local marketplace) now reference `~/projects/council` and `council@council` (or equivalent). If the text still reads awkwardly after the substitution, adjust the surrounding wording for coherence.

---

## Task 6: Rewrite `README.md` install section

**Files:**
- Modify: `/home/dylan/projects/council/README.md`

The current install section is:

```markdown
## Install

\`\`\`
/plugin install council
\`\`\`
```

This one-liner does not work — it assumes Council is in a default marketplace, which it isn't. The replacement describes the flow that the user actually uses successfully: add the marketplace first (by local path during dev, or by git URL once published), then install Council from the `/plugin` GUI.

- [ ] **Step 1: Replace the install section**

Use the Edit tool:
- `file_path`: `/home/dylan/projects/council/README.md`
- `old_string`:
```
## Install

```
/plugin install council
```

```
- `new_string`:
```
## Install

Council is distributed as a Claude Code plugin marketplace. To install:

1. In Claude Code, run `/plugin` to open the plugin GUI.
2. Choose **Add marketplace** and paste the repo URL: `https://github.com/dylanfetch/council` (or a local clone path during development).
3. From the marketplace list, select **council** and install it.

Once installed, the three skills (`/create-persona`, `/create-council`, `/ask-council`) become available in any Claude Code session.

```

- [ ] **Step 2: Verify**

Read `/home/dylan/projects/council/README.md`. Confirm the install section now contains the three-step flow above and the broken `/plugin install council` one-liner is gone. Also confirm that the rest of the README still flows naturally into the Quick Start section below.

- [ ] **Step 3: Scan for other stale references**

Grep `/home/dylan/projects/council/README.md` for:
- `agent-conference-experimental` — expected: zero matches
- `dylan-local` — expected: zero matches
- `council-local` — expected: zero matches

If any match is found, fix it and re-verify.

---

## Task 7: Write `LICENSE` (MIT)

**Files:**
- Create: `/home/dylan/projects/council/LICENSE`

- [ ] **Step 1: Write the LICENSE file**

Use the Write tool to create `/home/dylan/projects/council/LICENSE` with exactly this content (standard MIT template):

```
MIT License

Copyright (c) 2026 Dylan Fetch

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OF OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Verify**

Read the first three lines of `/home/dylan/projects/council/LICENSE`. Expected exactly:
```
MIT License

Copyright (c) 2026 Dylan Fetch
```

---

## Task 8: Final pre-commit review

**Files:**
- Read: all 18 files now in `/home/dylan/projects/council/`

- [ ] **Step 1: Confirm the full file manifest**

Run: `find /home/dylan/projects/council -type f -not -path '*/.git/*' | sort`
Expected exactly these 17 files (the 16 from Task 2 Step 6, plus `LICENSE`):
```
/home/dylan/projects/council/.claude-plugin/marketplace.json
/home/dylan/projects/council/.gitignore
/home/dylan/projects/council/CLAUDE.md
/home/dylan/projects/council/IDEAS.md
/home/dylan/projects/council/LICENSE
/home/dylan/projects/council/README.md
/home/dylan/projects/council/council/.claude-plugin/plugin.json
/home/dylan/projects/council/council/skills/ask-council/SKILL.md
/home/dylan/projects/council/council/skills/ask-council/styles/roundtable.md
/home/dylan/projects/council/council/skills/ask-council/styles/telephone.md
/home/dylan/projects/council/council/skills/create-council/SKILL.md
/home/dylan/projects/council/council/skills/create-persona/SKILL.md
/home/dylan/projects/council/council/templates/persona.md
/home/dylan/projects/council/docs/superpowers/plans/2026-04-09-council-plugin.md
/home/dylan/projects/council/docs/superpowers/plans/2026-04-10-council-repo-extraction.md
/home/dylan/projects/council/docs/superpowers/specs/2026-04-09-council-plugin-design.md
/home/dylan/projects/council/docs/superpowers/specs/2026-04-10-council-repo-extraction-design.md
```

- [ ] **Step 2: Confirm no stale references exist anywhere in the new repo**

Run three Grep checks over the entire `/home/dylan/projects/council/` tree (excluding `.git/`):
- Pattern: `agent-conference-experimental` — expected: zero matches
- Pattern: `dylan-local` — expected: zero matches
- Pattern: `council-local` — expected: zero matches

If any of these return matches, diagnose which file, fix it, and re-verify.

- [ ] **Step 3: Check `git status` — see what would be staged**

Run: `git -C /home/dylan/projects/council status --short`
Expected: every file from Step 1 shown as `??` (untracked). No `!` (ignored) entries except possibly nothing.

- [ ] **Step 4: Dry-run `git add` to see what it would include**

Run: `git -C /home/dylan/projects/council add --dry-run .`
Expected: a line for every file in the manifest. Critically: NO `docs/` files should appear as ignored. If any `docs/` files are missing from the output, Task 3 was not applied correctly — go back and fix `.gitignore`.

---

## Task 9: Initial commit

**Files:**
- Stage: everything in `/home/dylan/projects/council/`

- [ ] **Step 1: Stage all files**

Run: `git -C /home/dylan/projects/council add .`
Expected: no output.

- [ ] **Step 2: Verify staged manifest**

Run: `git -C /home/dylan/projects/council diff --cached --stat`
Expected: 17 files changed, all insertions, including all four `docs/superpowers/` files.

- [ ] **Step 3: Create the initial commit**

Run:
```bash
git -C /home/dylan/projects/council commit -m "$(cat <<'EOF'
Initial commit: Council plugin for Claude Code

Council is a Claude Code plugin for assembling councils of AI advisor
personas and consulting them using structured deliberation styles.
Includes three skills (create-persona, create-council, ask-council),
two deliberation styles (telephone, roundtable), a persona template,
and the design spec + implementation plan that produced this repo.

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
EOF
)"
```
Expected: `[main (root-commit) <sha>] Initial commit: Council plugin for Claude Code` followed by file-count summary.

- [ ] **Step 4: Verify the commit**

Run: `git -C /home/dylan/projects/council log --oneline && git -C /home/dylan/projects/council status`
Expected: one commit on `main`, working tree clean.

---

## Task 10: Add remote and push to GitHub

**Files:**
- Configure: git remote `origin` pointing at `$GITHUB_URL`

**Prerequisite:** User has created an empty public repo at `github.com/dylanfetch/council` (or `/agent-council`) and provided the clone URL.

- [ ] **Step 1: Add the remote**

Run: `git -C /home/dylan/projects/council remote add origin <GITHUB_URL>`
(Substitute the URL the user provided, e.g., `https://github.com/dylanfetch/council.git`.)
Expected: no output.

- [ ] **Step 2: Verify the remote**

Run: `git -C /home/dylan/projects/council remote -v`
Expected: two lines, both showing `origin` with the user-provided URL (fetch and push).

- [ ] **Step 3: Push `main` and set upstream tracking**

Run: `git -C /home/dylan/projects/council push -u origin main`
Expected: successful push, output ending with `branch 'main' set up to track 'origin/main'`.

If push fails with authentication error, halt and ask the user to resolve (they may need to set up a credential helper or provide a token).

- [ ] **Step 4: Verify the push landed**

Run: `git -C /home/dylan/projects/council log --oneline origin/main`
Expected: one commit matching the local `main` HEAD.

Additionally: fetch the repo page via curl to confirm it's publicly visible.
Run: `curl -sS -o /dev/null -w "%{http_code}\n" https://github.com/dylanfetch/council`
(Substitute fallback name if applicable.)
Expected: `200`.

---

## Task 11: Verify source worktree is untouched

**Files:**
- Read only: `/home/dylan/projects/agent-conference-experimental`, `/home/dylan/projects/agent-conference`

This is the safety-net verification. The spec promised both `agent-conference` and `agent-conference-experimental` would be untouched. Confirm.

- [ ] **Step 1: Confirm the experimental worktree has the same HEAD as the baseline captured in Task 1 Step 2**

Run: `git -C /home/dylan/projects/agent-conference-experimental log -1 --format='%H %s'`
Expected: exact match against the HEAD SHA recorded in Task 1 Step 2. Any difference means a stray commit landed on `experimental` during extraction — halt and investigate.

- [ ] **Step 2: Confirm the experimental worktree is clean**

Run: `git -C /home/dylan/projects/agent-conference-experimental status --short`
Expected: empty output.

- [ ] **Step 3: Confirm the original `agent-conference` repo is untouched**

Run: `git -C /home/dylan/projects/agent-conference log -1 --oneline && git -C /home/dylan/projects/agent-conference status --short`
Expected: same `3b18612 demo runs` HEAD, clean status.

- [ ] **Step 4: Report success**

Summarize for the user:
- New repo URL
- Commit SHA pushed
- Confirmation that the two source repos are unchanged
- Reminder that the next step is for the user to run the `/plugin` GUI install flow against the new repo (either via local path or via the published GitHub URL) to verify the plugin still works end-to-end.

---

## Success criteria

All of these must be true after Task 11:

- [ ] `https://github.com/dylanfetch/council` returns HTTP 200 and shows all 17 files
- [ ] The LICENSE file renders on GitHub with "MIT License" and "Copyright (c) 2026 Dylan Fetch"
- [ ] `git log` on the new repo shows exactly one commit
- [ ] The source `agent-conference-experimental` worktree has unchanged HEAD and clean status
- [ ] The source `agent-conference` master has unchanged HEAD and clean status
- [ ] Zero occurrences of `agent-conference-experimental`, `dylan-local`, or `council-local` anywhere in the new repo (excluding `.git/`)
- [ ] `.gitignore` in the new repo does NOT contain `docs/`
