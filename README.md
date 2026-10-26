# openspec-superpowers-schema

An [OpenSpec](https://github.com/Fission-AI/OpenSpec) workflow schema that runs each OpenSpec artifact step
through the matching [superpowers](https://github.com/obra/superpowers) skill.

## Install

### With your agent (recommended)

Run your agent in your project root and paste:

```
Install the OpenSpec schema from https://github.com/mrckndt/openspec-superpowers-schema by following the
"Manual install" steps in its README.md. Clone it to a temporary directory outside this project. Merge into
existing config.yaml, AGENTS.md, CLAUDE.md and .gitignore instead of overwriting them. If a requirement is
missing, tell me and stop; ask before running git init or committing.
```

### Requirements

- **OpenSpec CLI** (tested with 1.15.0): [installation](https://openspec.dev/docs/installation)
- **superpowers plugin** for your agent (tested with 7.0): [installation](https://github.com/obra/superpowers#installation).
  Install it for all projects, or for this one from the project root: a project install from a subfolder applies
  to that subfolder only. Restart your agent and check that its skills (e.g. `superpowers:brainstorming`) are listed.
- **Git repository** with at least one commit: each change gets its own worktree.

### Manual install

Run these from your project root.

1. Clone this repository outside your project (a clone inside it is a nested git repository):
   ```
   git clone https://github.com/mrckndt/openspec-superpowers-schema.git <clone-dir>
   ```
2. Initialize OpenSpec if you have not already. This generates the `/opsx:*` commands and skills for your tool:
   ```
   openspec init
   ```
3. Copy the schema into the project:
   ```
   mkdir -p openspec/schemas
   cp -R <clone-dir>/openspec/schemas/superpowers openspec/schemas/
   ```
4. Select it in `openspec/config.yaml`, replacing the `schema: spec-driven` that `openspec init` writes (create the
   file with just this line if it does not exist; every other key is optional):
   ```
   schema: superpowers
   ```
5. Add the rules from `<clone-dir>/AGENTS.md` to your project's `AGENTS.md`. Leave out its
   `Claude Code: worktree tools` section for other agents. For Claude Code, also add `@AGENTS.md` to your
   `CLAUDE.md` (or copy the clone's): Claude Code reads `AGENTS.md` on its own only when no `CLAUDE.md` exists
   in the project or above it.
6. Git-ignore the worktree directory (each change's worktree lives there):
   ```
   .worktrees/
   ```
7. Check that OpenSpec picks it up:
   ```
   openspec schema validate superpowers
   openspec schemas
   ```

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
