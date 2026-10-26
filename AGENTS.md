# OpenSpec + superpowers

OpenSpec owns the artifacts in `openspec/changes/<change-name>/`; superpowers owns the process. The schema
`openspec/schemas/superpowers/` names the superpowers skill for each artifact step.

- **Entry point:** feature work starts with `/opsx:explore`, then `/opsx:propose <change-name>`. Explore only thinks
  and reads: when the user wants to capture it as a change, run `/opsx:propose`. Invoke superpowers:brainstorming
  and superpowers:writing-plans only when an `openspec instructions` result says so.
- **IMPORTANT - output paths:** design doc: `openspec/changes/<change-name>/design.md`; plan:
  `openspec/changes/<change-name>/tasks.md`. Never create files under `docs/superpowers/`.
- **Worktree:** before `/opsx:propose`, without asking: pick the change name, then create the worktree via
  superpowers:using-git-worktrees in these steps:
  1. Record the output of `git branch --show-current` as the base branch. If `git check-ignore -q .worktrees`
     fails, ask the user before the skill commits a `.gitignore` change to the base branch.
  2. Use the skill's git fallback (Step 1b) with `.worktrees/` as its directory, also when a native worktree tool
     exists: `git worktree add .worktrees/<change-name> -b <change-name>`.
  3. Switch into the worktree and confirm `git branch --show-current` prints `<change-name>`.
  - Run `/opsx:apply` from the worktree: the change folder exists only on its branch. In a new session, switch
    into it first.
  - Untracked and git-ignored files, such as reference material, are not in the worktree: read them from the
    main checkout by absolute path.
- **Design:** brainstorming may skip it; `design.md` then holds one `Skipped: <reason>` line.
- **Missing skill:** if a superpowers skill this file names is unavailable, say so and stop; never substitute your
  own version of it. The schema checks the skills its steps invoke.
- **Plan approved:** commit the change folder, then stop. Execution starts only when the user runs
  `/opsx:apply`, which uses superpowers:subagent-driven-development, never inline execution.
- **If the user chooses a local merge when finishing:** first check the main checkout for uncommitted changes. If
  any touch files the branch changed, the merge would fail: show them to the user and ask how to handle them;
  never set them aside on your own.
- **Apply to finish:** the schema's `apply` instruction covers task ticking, archive, and finishing.

## Claude Code: worktree tools

- Switch into a worktree: `EnterWorktree` with its `path`.
- Before finishing removes the worktree: `ExitWorktree` with `keep`.
