# openspec-superpowers-schema

An [OpenSpec](https://github.com/Fission-AI/OpenSpec) workflow schema that runs OpenSpec's design, plan and build
steps through the matching [superpowers](https://github.com/obra/superpowers) skills.

## What it is and why

- **[OpenSpec](https://github.com/Fission-AI/OpenSpec) brings spec-driven development (the what).** Before any code
  exists, you and the agent agree in writing on why a change is needed and what "done" means, as testable
  requirements in `openspec/changes/<change-name>/`. You review words instead of code, and on archive the specs
  are merged into `openspec/specs/`, which stays a current description of what your system does.
- **[superpowers](https://github.com/obra/superpowers) brings test-driven development (the how).** A short design
  discussion with you settles the approach, a plan breaks it into tasks, and subagents build them test-first, each
  task reviewed by a fresh reviewer.

Used together, the specs say what to build and the tests prove it was built. On their own the two don't know about
each other and write to different places (`docs/superpowers/` vs. the change folder). This schema and the rules in
`AGENTS.md` (copied into your project during install) hand the work from one to the other:

1. **OpenSpec writes the what:** `proposal.md` and `specs/`, on a branch of its own in `.worktrees/<change-name>/`.
2. **superpowers writes the how:** `design.md` and `tasks.md` (which `/opsx:propose` would normally write itself),
   into the same change folder. The design
   discussion (superpowers' brainstorming skill) starts from the proposal and specs, so it only asks the last
   questions. `/opsx:propose` stops at the approved plan; only `/opsx:apply` builds.
3. **superpowers builds:** test-first, task by task, each reviewed, then a whole-branch review against the specs and
   design. Unfixed Critical or Important findings, open tasks or failing tests stop the run before archive.
4. **OpenSpec archives:** the specs are merged into `openspec/specs/`. Every scenario has a test, so they stay the
   source of truth.

**The workflow** combines OpenSpec's [loop](https://openspec.dev/docs/quickstart#the-loop-at-a-glance) with
superpowers' [basic workflow](https://github.com/obra/superpowers#the-basic-workflow), in two views. What you type:

```
/opsx:explore → /opsx:propose → /opsx:apply
```

And the steps behind it, in order (the table below lists each step's artifact, command and skill):

```
explore → proposal → specs → design → tasks → build → archive
```

| Step | Artifact | Command | Skill |
|---|---|---|---|
| Explore | none, until you ask it to capture the change (or accept its offer) | `/opsx:explore` | - |
| Proposal (why and what) | `proposal.md` | `/opsx:propose` | - |
| Specs (behavior contract) | `specs/**/spec.md` | `/opsx:propose` | - |
| Design (how) | `design.md`, starting with `Skipped: <reason>` if skipped | `/opsx:propose` | `superpowers:brainstorming` |
| Plan/tasks | `tasks.md`, one checkbox per task | `/opsx:propose` | `superpowers:writing-plans` |
| Build | code, ticked `tasks.md` | `/opsx:apply` | `superpowers:subagent-driven-development` |
| Finish | archived change, specs merged into `openspec/specs/` | `/opsx:apply` (runs `openspec archive`) | `superpowers:finishing-a-development-branch` |

**Note:** if brainstorming changes scope or behavior, the agent updates `proposal.md` and `specs/`, validates the
change, and lists the edits when asking for design approval.

## Install

### Requirements

- **OpenSpec CLI** (tested with 1.15.0): [installation](https://openspec.dev/docs/installation)
- **superpowers plugin** for your agent (tested with 7.0): [installation](https://github.com/obra/superpowers#installation)
  - **Scope:** user scope covers every project, project scope only this one.
  - **Project scope:** run the install from the project root, after [Prepare the project](#prepare-the-project) if
    the project is new; from a subfolder it applies to that subfolder only.
  - **Check:** restart your agent; skills such as `superpowers:brainstorming` should appear in its skill list.

### Prepare the project

Each change runs in a git worktree, which sees only committed files. Start from a git repository with at least
one commit and OpenSpec initialized. For a new project:

```
mkdir my-project && cd my-project
git init && git commit --allow-empty -m "Initial commit"
openspec init
```

In an existing project, run only the steps it still lacks (git repository, one commit, `openspec init`).
`openspec init` asks which tool to set up and generates the `/opsx:*` commands of your profile (see
[Usage](#usage)).

### With your agent (recommended)

After Requirements and Prepare the project, run your agent in the project root and paste:

```
Install the OpenSpec schema from https://github.com/mrckndt/openspec-superpowers-schema by following the
"Manual install" steps in its README.md. First check that `openspec --version` works, that
`superpowers:brainstorming` is in your skill list (if not, tell me to restart the agent), and that this is a
git repository with at least one commit where `openspec init` has run; if anything is missing, tell me and
stop. Clone to a temporary directory outside this project. Append the rules from the clone's AGENTS.md to
AGENTS.md, omitting its `Claude Code` section unless you are Claude Code; if you are Claude Code, add only the
line `@AGENTS.md` to CLAUDE.md. Create a file only if it is missing, skip content that is already there, start a
new line when a file lacks a trailing newline, and never replace other content. Set `schema: superpowers` in
openspec/config.yaml. Delete the clone, then show me `git status`. Once I confirm, commit the install files and
the `openspec init` output (`openspec/` and your tool's folder, e.g. `.claude/`), leaving my other changes
uncommitted.
```

### Manual install

Run these in one terminal from your project root, after [Prepare the project](#prepare-the-project).

1. Clone this repository outside your project (a clone inside it is a nested git repository):
   ```
   CLONE_DIR=$(mktemp -d)
   git clone https://github.com/mrckndt/openspec-superpowers-schema.git "$CLONE_DIR"
   ```
2. Copy the schema into the project:
   ```
   mkdir -p openspec/schemas
   cp -R "$CLONE_DIR/openspec/schemas/superpowers" openspec/schemas/
   ```
3. Select it in `openspec/config.yaml`, replacing the `schema: spec-driven` that `openspec init` writes (create the
   file with just this line if it does not exist; every other key is optional):
   ```
   schema: superpowers
   ```
4. Add the rules from `$CLONE_DIR/AGENTS.md` to your project's `AGENTS.md`, below any existing content: create the
   file if it is missing, never overwrite it, start a new line first if the last one has no trailing newline,
   and skip this if the file already has the heading `# OpenSpec + superpowers`.
   - **Claude Code:** keep the whole file, and add the line `@AGENTS.md` on its own line to `CLAUDE.md` (copy
     `$CLONE_DIR/CLAUDE.md` only if the project has none; skip the line if it is already there). Claude Code
     reads `AGENTS.md` on its own (v2.1.277 or later) only when no `CLAUDE.md` exists in the project or above
     it, so it needs this import whenever one does.
   - **Other agents:** leave out the `Claude Code` section.
5. Add this line to `.gitignore` (create the file if it is missing, skip it if the line is already there, and
   start a new line first if the last one has no trailing newline):
   ```
   .worktrees/
   ```
6. Check that OpenSpec picks it up. The output of the first command contains `Schema 'superpowers' is valid`,
   the second lists `superpowers (project)`:
   ```
   openspec schema validate superpowers
   openspec schemas
   ```
7. Delete the clone:
   ```
   rm -rf "$CLONE_DIR"
   ```
8. Commit the setup so each change's worktree contains it. Stage only the install files and the `openspec init`
   output. `git add` aborts and stages nothing if a path does not exist, so leave out what you do not have:
   ```
   git add openspec .gitignore AGENTS.md
   git add CLAUDE.md   # Claude Code only
   git add .claude     # the folder openspec init created for your tool, if any
   git commit -m "Set up OpenSpec with the superpowers schema"
   ```

## Usage

`/opsx:*` commands are typed into your agent's chat; `openspec` and `git` commands run in a terminal. The normal
flow is three commands:

```
/opsx:explore                 # think it through, read the code; writes nothing until you ask it to capture
/opsx:propose <change-name>   # worktree (per AGENTS.md), proposal, specs, then design and plan; stops at the approved plan
/opsx:apply <change-name>     # test-first build with reviews, archive, finish
```

- **Same session:** run `/opsx:propose` in the same session as the exploration; otherwise brainstorming has to ask
  again what you already settled.
- **Review the plan:** `/opsx:propose` stops with the plan written, before any code. Read `proposal.md`, then
  `specs/` (would you accept these requirements as done?), then `tasks.md`. Edit the files or tell the agent what
  is wrong; `/opsx:update` revises them coherently.
- **Capture shortcut:** instead of typing `/opsx:propose`, tell the agent in the explore session to capture the
  change (optionally naming it); with the rules from `AGENTS.md` it creates the worktree and runs `/opsx:propose`
  inside it.
- **Name:** `<change-name>` is optional on `/opsx:propose`; the agent derives one from your request.

**Continuing in a new session:** the approved plan is committed on the change's branch, in
`.worktrees/<change-name>/`, so closing the agent after `/opsx:propose` loses nothing. A new session starts in the
main checkout, where the change folder does not exist.

- **Claude Code:** start it in the project root and ask the agent to switch into the worktree, then run
  `/opsx:apply <change-name>`. Starting it inside the worktree breaks cleanup: finishing removes the session's
  working directory.
- **Other agents:** from the project root, switch into the worktree, then start your agent in that folder and run
  `/opsx:apply <change-name>`:

  ```
  git worktree list              # shows the path if you forgot it
  cd .worktrees/<change-name>
  ```

To finish an interrupted `/opsx:propose`, run it again with the same change name; it asks whether to continue the
existing change.

Run the `/opsx:*` commands and `openspec status`, `show` and `validate` from the change's worktree.

**Other OpenSpec commands** (`openspec init` generates only those in your profile; `/opsx:verify` needs the
expanded profile: `openspec config profile`, then `openspec update`):

| Command | Use it to |
|---|---|
| `/opsx:update <change-name>` | Revise a change's proposal, specs, design and plan after `/opsx:propose` stops; the agent confirms each edit with you and keeps them coherent |
| `/opsx:verify <change-name>` | Check the code against the change's specs and design (expanded profile) |
| `/opsx:sync <change-name>` | Merge a change's delta specs into `openspec/specs/` without archiving it |
| `/opsx:archive <change-name>` | Archive manually, e.g. after an apply run that stopped before finishing |

**CLI commands to inspect and validate a change:**

| Command | Use it to |
|---|---|
| `openspec status --change <change-name>` | See which artifacts are done and which step is next |
| `openspec list` / `openspec list --specs` | List changes in flight / specs merged so far |
| `openspec show <change-name>` | Read a change or spec in the terminal |
| `openspec validate <change-name>` | Check the artifacts after editing them by hand |
| `openspec view` | Open the interactive dashboard |

**superpowers skills outside the automated flow:**

| Skill | When |
|---|---|
| `superpowers:systematic-debugging` | A bug, failing test, or unexpected behavior, before proposing a fix |
| `superpowers:dispatching-parallel-agents` | Two or more independent problems, e.g. unrelated test failures, worked on in parallel |
| `superpowers:test-driven-development` | Small fixes you make without a change folder |
| `superpowers:verification-before-completion` | Before claiming something works or is done |
| `superpowers:requesting-code-review` | A review of work done outside `/opsx:apply` |
| `superpowers:receiving-code-review` | Acting on review feedback from people or tools |
| `superpowers:finishing-a-development-branch` | Merge, PR, or keep a branch that `/opsx:apply` did not finish |
| `superpowers:diagnosing-superpowers` | A session went wrong and you want to know why |

Leave `superpowers:brainstorming`, `superpowers:writing-plans` and `superpowers:subagent-driven-development` to the
schema: invoked directly, they bypass the change folder, its validation, and the apply rules.

## Known limitations

- The superpowers skills still bring their own defaults (`docs/superpowers/` paths, a choice of execution method,
  a plan header that points to `executing-plans`). The schema instructions and the rules in `AGENTS.md` override
  them; if either is missing, the defaults return.

## License

MIT
