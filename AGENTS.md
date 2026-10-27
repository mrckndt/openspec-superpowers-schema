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
     If it succeeds, skip the skill's own ignore check and `.gitignore` commit: that check omits the slash.
  2. Use the skill's git fallback (Step 1b) with `.worktrees/` as its directory, also when a native worktree tool
     exists: `git worktree add .worktrees/<change-name> -b <change-name>`.
  3. Switch into the worktree and confirm `git branch --show-current` prints `<change-name>`. If your shell does
     not keep `cd` between commands, run every command as `cd <worktree> && ...`, or ask the user to restart the
     agent inside the worktree.
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
- **Local merge when finishing:** shell variables do not survive between commands. In finishing's Step 2, print
  `GIT_DIR`, `GIT_COMMON` and `WORKTREE_PATH`, note the values, and use them in Step 6; never recompute them
  after leaving the worktree. `<main-checkout>` is the first path `git worktree list` prints. Before merging:
  1. Run `git -C <worktree> status --porcelain`. If it lists files, show them to the user and ask whether to
     commit them to `<change-name>` first: uncommitted files are not part of the merge.
  2. Run `git -C <main-checkout> status --porcelain` and `git diff --name-only <base-branch>...<change-name>`.
     If a path appears in both, show those paths to the user and ask how to handle them; leave the files as
     they are until the user answers.
  3. If the base branch has no upstream, skip finishing's `git pull`.
  - Run each main-checkout command of Step 5 and Step 6 as `git -C <main-checkout> ...`, including
    `git -C <main-checkout> worktree remove <WORKTREE_PATH>`.

## Claude Code

- **Switch into a worktree:** `EnterWorktree` with its `path`.
- **Before finishing's Step 6:** after the merged result passes its tests, if this session called `EnterWorktree`,
  call `ExitWorktree` with `keep`. Call it no earlier: Step 2 must run inside the worktree.
- **New session on a change:** start Claude Code in the main checkout and switch with `EnterWorktree`. A session
  started inside `.worktrees/<change-name>` loses its working directory when finishing removes the worktree.
- **Subagent models:** never above Opus (such as Fable). Read each superpowers model choice that goes higher
  ("most capable available model", "a tier above", "a more capable model") as Opus. Name the model on every
  dispatch: an omitted one inherits the session's.
