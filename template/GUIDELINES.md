# Guidelines for CLAUDE.md and the session handoff

Use these for every new project. The templates in this folder follow them.

## Four files, four jobs

| File | Job | Changes | Read |
|---|---|---|---|
| `CLAUDE.md` (repo root) | **How work gets done**: rules, commands, workflow, model routing | Rarely | Automatically, every session |
| `docs/SESSION_HANDOFF.md` | **Where the project is now**: state, decisions in force, open items | Every session | First thing, every session |
| `docs/IDEAS.md` | **Ideas ledger**: one line per idea tried, adopted, rejected or open | When an idea is added or tested | Searched (not read) before proposing or testing an idea |
| `docs/HANDOFF_ARCHIVE.md` | **History**: full write-ups of experiments and decisions, finished work | Appended to | Only when a task needs the history |

Plus four subagents in `.claude/agents/`: `planner.md` (Opus, high effort, plans changes before they're built), `explorer.md` (Haiku, low effort, read-only search), `reviewer.md` (Opus, high effort, reviews a finished branch; Sonnet for docs-only branches) and `analyst.md` (Opus, high effort, designs experiments and judges results; delete it if the project has none). `.claude/settings.json` sets the main session to Sonnet at medium effort, blocks reading of files that waste context, and wires the after-edit hook. `.claude/rules/hard-area.md` is a template for rules that load only when one area's files are opened.

## Skills hold the process; CLAUDE.md holds the specifics

- `/release` and `/handoff` are the same for every project. Each project's `CLAUDE.md` has a "Release and handoff
  specifics" section with its file paths, metric, mirror location, build check and deploy target; the skills read it.
- Every improvement to how work gets done is made in two places: in the project, and generalized here (templates,
  skills, this guide). A project-only fix that could apply elsewhere is half done.

## Changing hosts

Hosting changes over a project's life, so the deploy target lives in one place: the **Deploy** line of `CLAUDE.md`'s
"Release and handoff specifics". Nothing else in the workflow names a host. When a project moves:

1. Set up the new host and confirm a deploy works there before touching the old one.
2. Update the **Deploy** line, and the handoff's current state, to the new host.
3. Stop the old host from building. Prefer a setting in the repo (for example a config whose build check always skips)
   over deleting the config, which can make the old host build again; also stop builds in the old host's own settings.
4. Remove code that only the old host used, once the new host does the same job.
5. Add an `IDEAS.md` row for the move, and say where the old site still serves until it is deleted there.

## Choosing models and effort

Pin judgment work to subagents and keep the main session on one model. Each subagent's file sets its model and effort,
and both the Claude app and Claude Code honor them, so the right effort is picked without anyone touching a setting.

- **Main session:** Sonnet at medium. In Claude Code, `.claude/settings.json` sets it (`"model": "sonnet"`,
  `"effortLevel": "medium"`); in the Claude app, keep the model picker on Sonnet.
- **Planner** (Opus, high): plans anything bigger than a small fix and every change in a hard area.
- **Reviewer** (Opus, high): the final review of every branch that changes code; dispatched on Sonnet only for
  branches that change nothing but docs, config or wording. One review per branch is rare and catches the bugs that
  don't crash, so it's where Opus pays most.
- **Analyst** (Opus, high), **explorer** (Haiku, low): as their files say.

Why not `opusplan` (Opus in plan mode, Sonnet otherwise): it uses one effort level for both, so Opus plans at medium
unless someone raises it by hand, and every plan-mode toggle is a model switch that re-reads the whole conversation
uncached. A planner subagent gets high effort automatically and leaves the main session's cache alone. Its cost: it
doesn't see the conversation, so the main session has to brief it well.

Escalate the main session to Opus only for a bug Sonnet has failed twice or a long stretch of hard-area edits, and
switch back after. Leaving the main session on Opus for routine work costs about twice as much.

**Checking the setup in a new session:** ask "which model are you?" (Sonnet); ask Claude to open a blocked file (it
should be refused); edit a tested file (the tests should run). The status line shows only in Claude Code's terminal;
the VS Code extension shows context and the cache clock in the prompt box instead.

## Working with the superpowers skills

The superpowers plugin (brainstorming, writing-plans, executing-plans, test-driven-development, systematic-debugging,
verification-before-completion, code review, worktrees) fits this setup. It defers to `CLAUDE.md`, so the
"Superpowers skills" section there decides how it's used. Three rules keep the two from doubling up:

1. **Size the process to the task.** Small fixes skip brainstorming, plan files and worktrees, but keep test-first and
   verification. Medium tasks use the planner's short plan. Only big features or hard-area changes run the full flow.
   A full feature cycle reads about 20,000 tokens of skill text, worth it for a feature and wasted on a tooltip.
2. **One planner, one reviewer.** The `planner` subagent (Opus) does writing-plans' thinking and the main session
   saves the file; the `reviewer` subagent runs any review a skill asks for, with that skill's template. No second
   planner or reviewer on the same work.
3. **Inline execution by default.** executing-plans runs well on a Sonnet main session. subagent-driven-development
   starts a fresh implementer and reviewer per task: use it only for independent tasks, with explicit models.

Worktrees are safe: the after-edit hook runs the tests in the checkout being edited, not the main one.

## Size budgets

Every session pays to read `CLAUDE.md` and the handoff before doing any work, on whatever model you are using.

- `CLAUDE.md`: about **100 lines**.
- `SESSION_HANDOFF.md`: about **150 lines / 8 KB** (roughly 2,000 tokens).
- `IDEAS.md`: one line per idea, no total limit. It is searched with the explorer subagent, not read whole.
- `HANDOFF_ARCHIVE.md`: no limit, because nothing reads it by default.

For scale: an 85 KB handoff costs about 20,000 tokens at the start of every session.

## What goes where

- A rule about *how* to work (push only when told, test first, branch names) → `CLAUDE.md`.
- A command (test, build, deploy) → `CLAUDE.md`.
- Which model to use for which kind of task → `CLAUDE.md`.
- What the project is, what is live, what is in progress → handoff.
- A decision that is still in force → handoff, one line with its date.
- The reasoning, numbers and runs behind a decision → archive. The handoff line points to it.
- A decision that was replaced → archive.
- An idea, feature or experiment (tried or not) → one row in `IDEAS.md`; its full write-up → archive.
- A finished open item → delete it from the handoff; its `IDEAS.md` row now says Adopted or Rejected.
- Anything already written elsewhere (a spec, a README, the code) → a link, never a copy.

## Writing rules

1. **Write the current truth, not the story.** "Defaults: Season, Average, Playoff boost Off" beats a paragraph about how you got there.
2. **Replace, don't append.** When something changes, edit the line in place. Appending is what makes handoffs balloon.
3. **One line per decision:** `YYYY-MM-DD: decision, short reason (archive: <section>)`.
4. **Keep only the numbers that matter.** A result goes in the handoff only if it decides what to do next; tables of runs go to the archive.
5. **No duplication between files.** If it is in `CLAUDE.md`, the handoff doesn't repeat it.
6. **Name the hard areas.** Listing the parts of the code where mistakes are subtle (core math, data model, auth, payments) tells future sessions when to use Opus and the Opus reviewer.

## Tracking ideas

- **Before proposing or testing an idea,** the explorer subagent searches `IDEAS.md` (and the archive if needed) for related keywords and reports matches with their status. A Rejected idea is re-tested only when something changed: new data, a different format, a bug found in the old test. Say what changed.
- **After a test,** edit the idea's row in place (status and the deciding numbers), add the full write-up to the archive, and if it was adopted, update "Decisions in force" in the handoff.
- **A new idea** gets the next number and status Open.
- **Rejected rows are the most valuable ones.** They stop future sessions from paying for the same test twice, so never delete them.

## Keeping context lean

Three guards in `.claude/` stop tokens being spent on things that never needed reading. Set them up when the project
has its first real files, and revisit when large files appear.

1. **Deny-read rules** (`settings.json` → `permissions.deny`). Block Claude from reading files that are big and rarely
   useful in full: data files, logs, experiment results, spreadsheets, generated or built files, dependencies, lock
   files. One accidental read of a large file can cost more than a whole session.
   - Syntax: `Read(/path)` anchors at the project root, `Read(name)` matches that name at any depth, `**` crosses folders.
   - A Read deny also blocks Claude's Edit and Write on the same path, which usefully stops hand edits to built files.
   - Scripts and programs still read the files; only Claude's own reading is blocked.
2. **Path-scoped rules** (`.claude/rules/*.md` with a `paths:` list). Rules that matter for one area only (the hard
   area, the UI, the API) load when Claude opens a matching file, instead of in every session. Move such rules out of
   `CLAUDE.md` and the handoff. Rules without `paths:` load every session, like `CLAUDE.md`.
3. **After-edit hook** (`.claude/hooks/after-edit.mjs`, wired in `settings.json`). Runs the unit tests after an edit to
   tested code and reports only failures, so breakage is caught on the edit that caused it. It also warns when the
   handoff passes its budget. Set `testOn` and `testCmd` at the top of the file; keep it to tests that run in seconds.

## When to release, compact or clear

- **Release** (commit, push, deploy) when a unit of work is finished and verified: a feature, a fix, an idea that changed status. Never mid-feature, never before an experiment is confirmed.
- **Clear** (`/clear`) at every task boundary, when switching to unrelated work, while waiting on a long-running job, and when coming back after more than an hour (the cache has expired). Write state down first.
- **Compact** (`/compact <what to keep>`) only mid-task in a long chat whose details you still need.
- **Rewind** (`/rewind`) instead of compacting when the chat went down a wrong path.
- After editing `CLAUDE.md`, clear before relying on the change; it loads only on `/clear`, `/compact` or a restart.
- Never clear with results that aren't recorded in the handoff or `IDEAS.md`.

### Status line cues

`.claude/statusline.mjs` shows model and effort, context used, the 5-hour plan window, the prompt cache and the git
branch, plus at most one hint. It runs locally and uses no tokens.

| It shows | Meaning | Do |
| --- | --- | --- |
| `ctx` green (under 50%) | Plenty of room | Keep going |
| `ctx` yellow (50-69%) + "/handoff, then /clear at the next stop" | Past half; cost per message and instruction-following get worse from here | Finish the current step, `/handoff` (or `/release`), `/clear` |
| `ctx` red (70% or more) | Close to auto-compaction, which you don't control | Stop at the next safe point and do the same now |
| "cache cold" + "/handoff, then /clear" (context 20% or more) | The cache expired; the next message re-reads everything uncached | `/handoff`, `/clear`, start fresh rather than continue |
| `5h` red (80% or more) + "near the 5-hour limit" | The plan window is nearly used up | `/handoff` now, so nothing is lost if the limit hits mid-task |

Thresholds are constants at the top of the script.

## End-of-session routine

1. Update the handoff: state, open items, "Next session starts with".
2. Update `IDEAS.md` rows for anything tested or shipped; move finished items and replaced decisions to the archive.
3. Check the size. If the handoff is over budget, archive the oldest material until it fits.
4. Change `CLAUDE.md` only when a rule, command or model-routing choice changed.

The `/handoff` skill runs this routine the same way every time, without pushing.
