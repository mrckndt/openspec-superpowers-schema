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
| Why and what | `proposal.md` | OpenSpec instruction |
| Behavior contract | `specs/**/spec.md` | OpenSpec instruction |
| How | `design.md` (skipped when nothing is open) | superpowers:brainstorming |
| Plan | `tasks.md` (one checkbox per task) | superpowers:writing-plans |
| Build | `/opsx:apply` | superpowers:subagent-driven-development |
| Finish | `openspec archive`, merge or PR | superpowers:finishing-a-development-branch |

## Why

- Specs stay the source of truth: every scenario is named in at least one test step of the plan, and archive
  merges the deltas into `openspec/specs/`.
- Design happens when it is needed: brainstorming decides whether the request leaves anything open, and skips
  `design.md` when it does not or when you ask it to just get going.
- Planning and building are separate: `/opsx:propose` stops at an approved, committed plan; nothing is built
  until you run `/opsx:apply`.
- Execution is reviewed: each task goes to a fresh implementer subagent and a reviewer, then a final
  whole-branch review, never inline.

## Install

Requirements: the OpenSpec CLI (tested with 1.14.3) and the superpowers plugin (tested with 7.0) for your agent.

1. Initialize OpenSpec in your project if you have not already. This generates the `/opsx:*` commands and
   skills for your tool:
   ```
   openspec init
   ```
2. Copy the schema into the project:
   ```
   cp -R openspec/schemas/superpowers <your-project>/openspec/schemas/
   ```
3. Select it in `<your-project>/openspec/config.yaml`:
   ```
   schema: superpowers
   ```
   If the file does not exist, create it with just that line; every other key in it is optional.
4. Add the rules from `AGENTS.md` to your project's `AGENTS.md`. For Claude Code, copy `CLAUDE.md` too, or
   add its one line, `@AGENTS.md`, to your existing `CLAUDE.md`: Claude Code reads `AGENTS.md` on its own only
   when no `CLAUDE.md` exists in the project or above it.
5. Git-ignore the directory your worktrees live in, for example:
   ```
   .claude/worktrees/
   ```
6. Check that OpenSpec picks it up:
   ```
   openspec schema validate superpowers
   openspec schemas
   ```

## Usage

```
/opsx:propose <change-name>   # worktree, proposal, specs, design, plan; stops at the approved plan
/opsx:apply <change-name>     # subagent-driven build, archive, finish
```

## Known limitations

- When `design.md` is skipped, `openspec status` still lists `design` as open and suggests it as the next
  step. The workflow continues regardless: `tasks` and `apply` do not wait for it.
- The superpowers skills keep their own defaults (`docs/superpowers/` paths, an execution-method question,
  a plan header naming executing-plans). The schema instructions and `AGENTS.md` override them; drop either
  and the defaults come back.

## License

MIT
