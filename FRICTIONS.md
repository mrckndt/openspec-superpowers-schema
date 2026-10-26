# Frictions from a test run

Test run: gogrep change `grep-only-llm-options`, 2026-10-26, Claude Code (Opus 5.5), OpenSpec CLI 1.14.3,
superpowers 7.0.0. Flow: `/opsx:explore` → capture steps from `AGENTS.md` → worktree → `/opsx:propose` →
brainstorming → writing-plans. `/opsx:apply` not run yet.

## 1. `git check-ignore -q .worktrees` gives a false negative (bug, AGENTS.md)

**Status:** fixed in `AGENTS.md` (check `.worktrees/` with the trailing slash). Still open upstream in
superpowers:using-git-worktrees.

**Where:** `AGENTS.md`, Worktree, step 1:

> If `git check-ignore -q .worktrees` fails, ask the user before the skill commits a `.gitignore` change to the base
> branch.

**What happens:** the README tells users to ignore `.worktrees/` (with a trailing slash). A pattern with a trailing
slash matches directories only. Before the first worktree exists, `.worktrees` is not on disk, so git cannot know it is
a directory, and the check fails although the directory is ignored:

```
$ cat .gitignore
.worktrees/
$ git check-ignore -q .worktrees; echo $?
1
$ git check-ignore -v .worktrees/
.gitignore:5:.worktrees/	.worktrees/
```

The agent then asks the user for permission to commit a `.gitignore` change that is not needed, or, with a less
careful agent, commits a duplicate entry. The first change in every project hits this, since `.worktrees/` does not
exist yet.

**Fix:** check with a trailing slash, which git reads as a directory whether it exists or not:

```
If `git check-ignore -q .worktrees/` fails, ...
```

The same check is in superpowers:using-git-worktrees (`git check-ignore -q .worktrees 2>/dev/null || git check-ignore
-q worktrees 2>/dev/null`); worth reporting upstream there too.

## 2. Decisions made in explore are not visible as decisions in the proposal

**Status:** fixed: the proposal template has `## Decided` and `## Assumptions`, the proposal instruction says what
goes in each, and the design instruction has brainstorming play the Decided list back first.

Explore settled several things with the user (for example an output limit, answered with a one-word "2"). The proposal
step wrote them into What Changes or as Assumptions, and brainstorming then opened with a question about one of them;
the user no longer remembered agreeing to it and had to ask what was meant.

**Suggestion:** the proposal instruction (or `AGENTS.md`, Capturing a change) separates "Decided with the user in
explore" from "Assumptions", and the design instruction has brainstorming play the decided list back in a few lines
before its first question, so the user can confirm or revise it.

## 3. Large scope changes in the design step mean several rewrites of proposal and specs

The user widened the scope twice during brainstorming (fix existing differences from GNU grep; then binary detection,
Unicode, warnings). The design instruction's "update proposal.md and specs/ first" worked, and validation passed every
time, but proposal and specs were rewritten three times. Not a bug: for larger changes the design step is where scope
is really settled. A one-line hint in the design instruction could help: after a scope change, re-run
`openspec validate --strict` and tell the user which requirements were added or changed.

## 4. OpenSpec CLI reports `spec-driven` while the change uses `superpowers` (upstream, cosmetic)

**Status:** the progress line is fixed upstream in OpenSpec 1.15.0 (#2120); README now says tested with 1.15.0.
`planningHome.defaultSchema` is by design: the built-in fallback for a project without `schema:`, not the schema in
use.

With `schema: superpowers` in `openspec/config.yaml`:

- `openspec new change <name>` prints `Creating change '<name>' with schema 'spec-driven'...`, then
  `Schema: superpowers`;
- `openspec status --json` shows `planningHome.defaultSchema: "spec-driven"` next to `schemaName: "superpowers"`.

`.openspec.yaml` correctly says `schema: superpowers`. Harmless, but it makes an agent or user doubt that the schema
was picked up. Belongs in an OpenSpec issue, not in this repository.

## 5. Builder check in brainstorming was worth it (keep)

Not a friction: the builder-check subagent found five real gaps in the design (Unicode classes, binary detection with
invalid UTF-8, GNU's pattern warnings, error texts, inner `\<`/`\>`). Worth keeping as it is.

## Not schema-related

Claude Code's worktree isolation blocks compound shell commands (Docker with `sh`, `sed` with variables, heredocs,
`git -C` to the main checkout) and writes outside the worktree in a worktree session. Workarounds: script files in the
scratchpad, single edits, `ExitWorktree` with `keep` to work elsewhere.
