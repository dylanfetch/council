# Extract Council plugin to its own public repository

**Date:** 2026-04-10
**Status:** Approved, ready for implementation planning

## Motivation

The Council plugin currently lives on the `experimental` branch of the [agent-conference](https://github.com/dylanfetch/agent-conference) repository as a worktree at [/home/dylan/projects/agent-conference-experimental](/home/dylan/projects/agent-conference-experimental). That branch has diverged completely from `master`: every pre-Council commit was wiped out at `8e8a496 feat: reimagine as Council`, and the five commits since then are all Council plugin work.

The plugin is installed, working, and ready to share. Keeping it buried inside an unrelated repo on a non-default branch makes it hard to discover, hard to install, and confusing to describe. Extracting it to its own public repository unblocks distribution and gives Council a clean identity of its own.

## Goals

- Council lives in a standalone public GitHub repository with a clean, presentable first impression.
- The new repository is installable via the same `/plugin` GUI flow the user already uses successfully (add marketplace by path → pick plugin → install).
- The original `agent-conference` repository and its `experimental` worktree are left completely untouched — no cleanup, no branch deletion, no remote changes.
- The extraction is a fresh start: one initial commit on the new repo, no preserved history.

## Non-goals

- **No feature work.** Items from [IDEAS.md](../../../IDEAS.md) are not touched in this session. They travel to the new repo as-is.
- **No structural redesign of the plugin itself.** The `skills/` layout, style files, and templates are copied byte-for-byte. Any improvements happen in follow-up sessions inside the new repo.
- **No new install paths.** Finding a better install flow (git-URL install, community marketplace entry, etc.) is deferred to a later session. The verification step uses the currently-working GUI marketplace flow.
- **No cleanup of the old setup.** The `agent-conference` repo, its `experimental` branch, and the `agent-conference-experimental` worktree all remain exactly as they are today.
- **No git history preservation.** The five Council commits on `experimental` are not carried forward. The new repo starts with a single "Initial commit".

## Decisions (from brainstorming)

| # | Decision | Rationale |
|---|----------|-----------|
| 1 | Single-plugin repo (not multi-plugin marketplace) | Council is the product; future features evolve inside it, not as sibling plugins. |
| 2 | Fresh history — one initial commit | The design spec and implementation plan for Council itself already document the "how it was built" story. No audit value in preserving the five commits. |
| 3 | Keep the marketplace wrapper layout (`.claude-plugin/marketplace.json` + `./council/` subdirectory) | The flat "plugin at repo root" layout was tried and did not work. The marketplace wrapper is the structure that is known to install successfully. |
| 4 | Repo name: `council`, fallback `agent-council` if taken | `council` is the cleanest. `agent-council` is the agreed fallback if `dylanfetch/council` is already owned on GitHub. |
| 5 | Public from creation | No secrets, no code, nothing to polish privately. User wants it public within hours. Private-then-public adds ceremony without protection. |
| 6 | Publish-then-verify | Create the GitHub repo and push the initial commit first, then run the install verification against the new location. User prioritized speed over a verification checkpoint. |
| 7 | Leave `agent-conference` and the `experimental` worktree completely alone | Belt-and-suspenders safety net. The source of truth remains available if anything is missed in the extraction. |

## Latent bugs discovered during spec review

These are broken in the current repo and must be fixed in the extraction:

- **`.gitignore` contains `docs/`.** The two existing design docs under `docs/superpowers/` are only in git because they were added before the ignore rule. New files under `docs/` silently fail to stage. The new repo's `.gitignore` must not ignore `docs/`.
- **`CLAUDE.md` references the marketplace as `dylan-local`, but the actual `.claude-plugin/marketplace.json` has `name: council-local`.** Both are wrong for a public repo — they need to become `council` — but flagging here so the find-and-replace is done against the correct strings.
- **`README.md` install instructions say `/plugin install council`, which does not work** without first adding a marketplace. The README needs a real rewrite of the install section, not a path substitution.

## Target repository layout

```
council/                                   (new repo root)
  .claude-plugin/
    marketplace.json                       (updated: name → "council")
  council/                                 (the plugin directory)
    .claude-plugin/
      plugin.json                          (unchanged)
    skills/
      create-persona/SKILL.md
      create-council/SKILL.md
      ask-council/
        SKILL.md
        styles/
          telephone.md
          roundtable.md
    templates/
      persona.md
  docs/
    superpowers/
      specs/
        2026-04-09-council-plugin-design.md
        2026-04-10-council-repo-extraction-design.md  (this document)
      plans/
        2026-04-09-council-plugin.md
        2026-04-10-council-repo-extraction.md         (created by writing-plans)
  .gitignore
  CLAUDE.md                                (updated: paths and marketplace name)
  IDEAS.md
  LICENSE                                  (new: MIT)
  README.md                                (updated: install instructions)
```

## File manifest

### Files copied verbatim

- `council/` — the entire plugin directory, recursively. Includes `.claude-plugin/plugin.json`, all `skills/`, and `templates/`.
- `IDEAS.md`
- `docs/superpowers/specs/2026-04-09-council-plugin-design.md`
- `docs/superpowers/plans/2026-04-09-council-plugin.md`
- `.gitignore`

### Files copied with edits

- **`.claude-plugin/marketplace.json`** — rename the marketplace `name` field from `council-local` to `council`. `owner.name` stays as `Dylan`. `plugins[0]` (name, source, description) is unchanged.
- **`.gitignore`** — remove the `docs/` line. This is a latent bug (see above); new repo should track `docs/` normally.
- **`CLAUDE.md`** — three kinds of edits:
  1. Replace `~/projects/agent-conference-experimental` with `~/projects/council` (or fallback name) wherever it appears.
  2. Replace `dylan-local` with `council` (even though the actual marketplace.json currently says `council-local`, the CLAUDE.md uses `dylan-local` — both stale variants get unified on `council`).
  3. Sanity re-read after replacement: ensure the "Installing locally for testing" section describes a working flow for the new repo.
- **`README.md`** — rewrite the install section. The current `/plugin install council` one-liner does not work without first adding a marketplace. The new install section should match the flow the user actually uses successfully: open `/plugin`, add a marketplace by pasting a path or git URL, pick Council from the GUI, install. Concrete copy to be drafted during implementation — this spec only commits to the requirement that the install instructions reflect a flow that actually works.

### Files copied from this extraction's brainstorming work

- `docs/superpowers/specs/2026-04-10-council-repo-extraction-design.md` — this document
- `docs/superpowers/plans/2026-04-10-council-repo-extraction.md` — the implementation plan produced next by `writing-plans`

These are copied so the new repo contains a complete record of its own origin.

### Files NOT copied

- `.git/` — fresh `git init`, no history carried over.
- `.claude/settings.local.json` — local harness settings, never belonged in git.
- `.council/` — runtime deliberation output, already gitignored.
- `.superpowers/` — local brainstorming session state.
- Any file or directory already matched by `.gitignore`.

### New files created in the new repo

- **`LICENSE`** — standard MIT license text, copyright year 2026, holder `Dylan` (matching the author field already used in both [council/.claude-plugin/plugin.json](council/.claude-plugin/plugin.json) and [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json)). If the user wants a fuller attribution (e.g., full name or GitHub handle), they can say so before the implementation commit — otherwise `Dylan` is the default.

## Updates required to transferred files

Four files are modified in flight:

1. **`.claude-plugin/marketplace.json`** — change top-level `name` from `council-local` to `council`. No other edits.
2. **`.gitignore`** — delete the `docs/` line.
3. **`CLAUDE.md`** — find-and-replace, then re-read to verify no broken references:
   - `~/projects/agent-conference-experimental` → `~/projects/council`
   - `agent-conference-experimental` → `council`
   - `dylan-local` → `council`
4. **`README.md`** — targeted rewrite of the install section to reflect the actually-working install flow (marketplace-add then pick-from-GUI). Rest of the file is left alone unless a broken reference is found on re-read.

Every edited file is re-read after editing to confirm no dangling references to `agent-conference-experimental`, `dylan-local`, or `council-local` remain.

## Execution outline

The `writing-plans` skill will produce the detailed implementation plan. At a high level:

1. Check GitHub availability of `dylanfetch/council`. Fall back to `dylanfetch/agent-council` if taken.
2. Create the local directory at `/home/dylan/projects/<repo-name>`.
3. `git init` with default branch `main`.
4. Copy files per the manifest above from [/home/dylan/projects/agent-conference-experimental](/home/dylan/projects/agent-conference-experimental).
5. Apply the in-flight updates: fix `marketplace.json` name, remove `docs/` from `.gitignore`, find-and-replace in `CLAUDE.md`, rewrite install section of `README.md`.
6. Write `LICENSE` (MIT).
7. `git add` + initial commit: `"Initial commit: Council plugin for Claude Code"`.
8. `gh repo create dylanfetch/<repo-name> --public --source=. --remote=origin --push`.
9. Confirm the GitHub repo page loads and the files are present.
10. Verification: in a separate Claude Code session, run the `/plugin` GUI install flow against the new local path, confirm the plugin installs and the skills are available. If verification fails, diagnose and fix in the new repo; the old worktree remains as a reference.

## Risks and rollback

- **Risk: broken install after push.** Mitigated by the verification step (#10). If the install fails, fix it in the new repo and push a follow-up commit. The public nature of the repo means the broken state is briefly visible, but since the repo is brand new with no users, the impact is effectively zero.
- **Risk: missed file during copy.** Mitigated by keeping the `agent-conference-experimental` worktree intact as the source of truth. Anything missed can be copied after the fact.
- **Risk: `dylanfetch/council` already taken on GitHub.** Mitigated by the agreed `agent-council` fallback.
- **Rollback:** If the extraction goes sideways, delete the new GitHub repo and the new local directory. The original `agent-conference` repo and its `experimental` worktree are untouched, so no rollback is needed on the source side.

## Success criteria

- `https://github.com/dylanfetch/council` (or `agent-council`) loads publicly with all expected files.
- `git clone` of the new repo and the `/plugin` GUI install flow produces a working Council plugin with all three skills (`create-persona`, `create-council`, `ask-council`) available.
- [/home/dylan/projects/agent-conference](/home/dylan/projects/agent-conference) and [/home/dylan/projects/agent-conference-experimental](/home/dylan/projects/agent-conference-experimental) are unchanged from their pre-extraction state (verified by `git status` and `git log` showing no new commits or branch changes).
