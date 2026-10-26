# OpenSpec + superpowers

OpenSpec owns the artifacts in `openspec/changes/<name>/`; superpowers owns the process. The schema
`openspec/schemas/superpowers/` names the superpowers skill for each artifact step.

- **Entry point:** feature work starts with `/opsx:explore`, then `/opsx:propose <name>`. Explore only thinks
  and reads: when the user wants to capture it as a change, run `/opsx:propose`. Invoke superpowers:brainstorming
  and superpowers:writing-plans only when an `openspec instructions` result says so.
- **IMPORTANT - output paths:** design doc: `openspec/changes/<name>/design.md`; plan:
  `openspec/changes/<name>/tasks.md`. Never create files under `docs/superpowers/`.
- **Worktree:** yes, without asking, via superpowers:using-git-worktrees before `/opsx:propose` (not at
  execution time). Pick the change name first and use it for the branch. Base the worktree on the current
  local branch, not `origin`: if your native worktree tool branches from `origin`, run
  `git worktree add <worktree-dir>/<name> -b <name>` instead and work in that directory.
- **Design:** conditional; the design artifact's instruction says when brainstorming skips it.
- **Plan approved:** commit the change folder, then stop. Execution starts only when the user runs
  `/opsx:apply`, which uses superpowers:subagent-driven-development, never inline execution.
- **Apply to finish:** the schema's `apply` instruction covers task ticking, archive, and finishing.
