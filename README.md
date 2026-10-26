# openspec-superpowers-schema

An [OpenSpec](https://github.com/Fission-AI/OpenSpec) workflow schema that runs each OpenSpec artifact step
through the matching [superpowers](https://github.com/obra/superpowers) skill.

## What it is

OpenSpec gives a change a durable home: `proposal.md`, delta specs, `design.md` and `tasks.md` under
`openspec/changes/<name>/`, validated by the CLI and merged into `openspec/specs/` on archive. superpowers gives
the agent a disciplined process: brainstorming before building, bite-sized TDD plans, and subagent-driven
execution with a review after every task.

On their own the two overlap and pull in different directions: superpowers writes its designs and plans to
`docs/superpowers/`, OpenSpec expects them in the change folder, and each has its own apply loop. This schema
and its `AGENTS.md` make them one workflow:

| Step | OpenSpec artifact | Done by |
|---|---|---|
| Why and what | `proposal.md` | `/opsx:propose` (or `/opsx:explore` when you ask it to capture the change) |
| Behavior contract | `specs/**/spec.md` | `/opsx:propose` (or `/opsx:explore` when you ask it to capture the change) |
| How | `design.md` (a `Skipped:` line when nothing is open) | `/opsx:propose` → superpowers:brainstorming |
| Plan | `tasks.md` (one checkbox per task) | `/opsx:propose` → superpowers:writing-plans |
| Build | code; checkboxes in `tasks.md` | `/opsx:apply` → superpowers:subagent-driven-development |
| Finish | `openspec archive` (moves the change to the archive, merges its specs), then merge or PR | `/opsx:apply` → OpenSpec CLI, then superpowers:finishing-a-development-branch |

## Why

- Specs stay the source of truth: every scenario is named in at least one test step of the plan, and archive
  merges the deltas into `openspec/specs/`.
- Design happens when it is needed: brainstorming decides whether the request leaves anything open, and skips
  the design only when you answer an explicit "Skip the design?" with yes.
- Planning and building are separate: `/opsx:propose` stops at an approved, committed plan; nothing is built
  until you run `/opsx:apply`.
- Execution is reviewed: each task goes to a fresh implementer subagent and a reviewer, then a final
  whole-branch review, never inline. Open review findings, unfinished tasks, or a failing test suite stop the
  run before archive and merge.

## Install

Requirements:

- The OpenSpec CLI (tested with 1.15.0): [installation](https://openspec.dev/docs/installation)
- The superpowers plugin for your agent (tested with 7.0): [installation](https://github.com/obra/superpowers#installation).
  Install it for all projects, or for this one from the project root: a project install from a subfolder applies
  to that subfolder only. Then restart your agent and check that its skills (e.g. `superpowers:brainstorming`)
  are listed
- A git repository with at least one commit: each change gets its own worktree.

Clone this repository outside your project; a clone inside it is a nested git repository.

1. Initialize OpenSpec in your project if you have not already. This generates the `/opsx:*` commands and
   skills for your tool:
   ```
   openspec init
   ```
2. Copy the schema into the project, from this repository:
   ```
   mkdir -p <your-project>/openspec/schemas
   cp -R openspec/schemas/superpowers <your-project>/openspec/schemas/
   ```
3. Select it in `<your-project>/openspec/config.yaml`, replacing the `schema: spec-driven` that
   `openspec init` writes:
   ```
   schema: superpowers
   ```
   If the file does not exist, create it with just that line; every other key in it is optional.
4. Add the rules from `AGENTS.md` to your project's `AGENTS.md`. Its `Claude Code: worktree tools` section is
   only for Claude Code; leave it out for other agents. For Claude Code, copy `CLAUDE.md` too, or
   add its one line, `@AGENTS.md`, to your existing `CLAUDE.md`: Claude Code reads `AGENTS.md` on its own only
   when no `CLAUDE.md` exists in the project or above it.
5. Git-ignore the worktree directory (`AGENTS.md` creates each change's worktree there):
   ```
   .worktrees/
   ```
6. Check that OpenSpec picks it up:
   ```
   openspec schema validate superpowers
   openspec schemas
   ```

Or, with this repository already on disk, run your agent in your project and paste:

```
Install the OpenSpec schema from <path-to-this-repo> by following the Install section of its README.md.
Merge into existing config.yaml, AGENTS.md, CLAUDE.md and .gitignore instead of overwriting them. If a
requirement is missing, tell me and stop; ask before running git init or committing.
```

## Usage

The normal flow is three commands:

```
/opsx:explore                 # think it through, read the code; writes nothing
/opsx:propose <change-name>   # worktree, proposal, specs, design, plan; stops at the approved plan
/opsx:apply <change-name>     # subagent-driven build, archive, finish
```

Run `/opsx:propose` in the same session as the exploration: brainstorming builds on that conversation instead of
asking again.

Other OpenSpec commands you may run yourself (which ones `openspec init` generates depends on your profile):

| Command | When |
|---|---|
| `/opsx:update <change-name>` | Revise a change's planning artifacts after review, keeping them coherent |
| `/opsx:sync <change-name>` | Merge a change's delta specs into `openspec/specs/` without archiving it |
| `/opsx:archive <change-name>` | Archive by hand, e.g. after an apply run that stopped before finishing |

superpowers skills that fit outside the automated flow:

| Skill | When |
|---|---|
| `superpowers:systematic-debugging` | A bug, failing test, or unexpected behavior, before proposing a fix |
| `superpowers:test-driven-development` | Small fixes you make without a change folder |
| `superpowers:verification-before-completion` | Before claiming something works or is done |
| `superpowers:requesting-code-review` | A review of work done outside `/opsx:apply` |
| `superpowers:receiving-code-review` | Acting on review feedback from people or tools |
| `superpowers:finishing-a-development-branch` | Merge, PR, or keep a branch that `/opsx:apply` did not finish |
| `superpowers:diagnosing-superpowers` | A session went wrong and you want to know why |

Leave `superpowers:brainstorming`, `superpowers:writing-plans` and `superpowers:subagent-driven-development` to the
schema: invoked directly, they bypass the change folder, its validation, and the apply rules.

## Known limitations

- The superpowers skills keep their own defaults (`docs/superpowers/` paths, an execution-method question,
  a plan header naming executing-plans). The schema instructions and `AGENTS.md` override them; drop either
  and the defaults come back.

## License

MIT
