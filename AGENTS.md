# OpenSpec + superpowers

OpenSpec owns the artifacts in `openspec/changes/<change-name>/`; superpowers owns the process. The schema
`openspec/schemas/superpowers/` names the superpowers skill for each artifact step.

- **Entry point:** feature work starts with `/opsx:explore`, then `/opsx:propose <change-name>`. Invoke
  superpowers:brainstorming and superpowers:writing-plans only when an `openspec instructions` result says so.
- **Capturing a change:** when the user wants to capture a request or an exploration as a change, also from inside
  `/opsx:explore`, do these steps in order:
  1. If the request leaves open who or what will use the result, ask that as one open question. Leave every other
     open question to brainstorming in the design step.
  2. Change name: the one the user gave; otherwise derive a kebab-case name from the request without asking.
  3. Create the worktree (below), then run `/opsx:propose <change-name>` inside it.
- **IMPORTANT - output paths:** design doc: `openspec/changes/<change-name>/design.md`; plan:
  `openspec/changes/<change-name>/tasks.md`. Never create files under `docs/superpowers/`.
- **Worktree:** create it via superpowers:using-git-worktrees, without asking, in these steps:
  1. Record the output of `git branch --show-current` as the base branch. If `git check-ignore -q .worktrees/`
     fails (keep the trailing slash: before the first worktree the directory does not exist, and without the
     slash git does not match the `.worktrees/` rule), ask the user before the skill commits a `.gitignore` change to the base branch.
  2. Use the skill's git fallback (Step 1b) with `.worktrees/` as its directory, also when a native worktree tool
     exists: `git worktree add .worktrees/<change-name> -b <change-name>`.
  3. Switch into the worktree and confirm `git branch --show-current` prints `<change-name>`.
  - Run `/opsx:apply` from the worktree: the change folder exists only on its branch. In a new session, switch
    into it first.
  - Untracked and git-ignored files, such as reference material, are not in the worktree: read them from the
    main checkout by absolute path.
- **Missing skill:** if a superpowers skill this file names is unavailable, say so and stop; never substitute your
  own version of it. The schema checks the skills its steps invoke.
- **Plan approved:** commit the change folder, then stop. Execution starts only when the user runs
  `/opsx:apply`, which uses superpowers:subagent-driven-development, never inline execution.
- **Apply after the last task:** when `/opsx:apply` reports all tasks done and the change folder is still in
  `openspec/changes/<change-name>/` (not archived), the CLI leaves out the schema's apply instruction. Read
  `apply.instruction` in `openspec/schemas/superpowers/schema.yaml` and continue from its final-review checks.
- **Local merge when finishing:** before merging, run `git -C <main-checkout> status --porcelain` and
  `git diff --name-only <base-branch>...<change-name>`, where `<main-checkout>` is the first path that
  `git worktree list` prints. If a path appears in both, show those paths to the user and
  ask how to handle them; leave the files as they are until the user answers. If the base branch has no
  upstream, skip finishing's `git pull`.

## Claude Code: worktree tools

- Switch into a worktree: `EnterWorktree` with its `path`.
- Before finishing removes the worktree: `ExitWorktree` with `keep`.
