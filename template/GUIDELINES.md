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
5. Once the old site is deleted on the old host, remove its config file too (nothing reads it any more).
6. Add an `IDEAS.md` row for the move, and say where the old site still serves until it is deleted there.

## Choosing models and effort

Pin judgment work to subagents and keep the main session on one model. Each subagent's file sets its model and effort,
and both the Claude app and Claude Code honor them, so the right effort is picked without anyone touching a setting.

- **Main session:** Sonnet at medium. In Claude Code, `.claude/settings.json` sets it (`"model": "sonnet"`,
  `"effortLevel": "medium"`); in the Claude app and the VS Code panel, keep the model picker on Sonnet: the picker overrides `settings.json`,
  so a session can silently run on Opus. Ask "which model are you?" at the start if in doubt.
- **Planner** (Opus, medium; dispatched with effort high for a hard area): plans anything bigger than a small fix and every change in a hard area.
- **Reviewer** (Opus, medium; effort high for a hard area): the final review of every branch that changes code; dispatched on Sonnet only for
  branches that change nothing but docs, config or wording. One review per branch is rare and catches the bugs that
  don't crash, so it's where Opus pays most.
- **Analyst** (Opus, high), **explorer** (Haiku, low): as their files say.

Why not `opusplan` (Opus in plan mode, Sonnet otherwise): it uses one effort level for both, so Opus plans at medium
unless someone raises it by hand, and every plan-mode toggle is a model switch that re-reads the whole conversation
uncached. A planner subagent gets its effort automatically and leaves the main session's cache alone. Its cost: it
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

4. **Skill budget.** using-superpowers says to invoke a skill at "even a 1% chance"; left alone, that loads
   test-driven-development and executing-plans on every small task (about 11% of a week's usage in one project).
   `CLAUDE.md` overrides it: those skills only for big features, and writing-plans / executing-plans never as
   subagents.

Worktrees are safe: the after-edit hook runs the tests in the checkout being edited, not the main one.

## Size budgets

Every session pays to read `CLAUDE.md` and the handoff before doing any work, on whatever model you are using.

- `CLAUDE.md`: about **100 lines**.
- `SESSION_HANDOFF.md`: aim for about **6 KB**; the after-edit hook warns past **150 lines or 8 KB** (roughly 2,000
  tokens). Bytes matter more than lines: a 113-line handoff was 12 KB because a few "lines" were paragraphs.
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
- Stable reference that changes rarely (the current model, defaults, architecture, data layout) → its own file
  (e.g. `docs/MODEL.md`) that the handoff points to in one line. It is read when a task needs it, not every session.
- **Trim by moving, not deleting.** Paste removed handoff lines unchanged into the archive under a dated "Handoff lines
  trimmed" heading, so nothing is lost and a later session can search for it. Only exact duplicates of `CLAUDE.md` may
  simply go.
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

### Permission prompts (`permissions.allow` / `ask`)

Prompt fatigue makes people approve everything blindly, so keep prompts only where they protect something. Rule of thumb:
ask for what is hard to undo or leaves the machine; allow the rest.

- **Allow:** read-only commands (many are auto-allowed already), routine local git (`add`, `commit`, `checkout`,
  `switch`, `stash`, `worktree add`), tests, and the exact build command. Local git is recoverable from the reflog.
- **Ask:** the template asks only for `git push`, merging (`gh pr merge`, the merge API) and deleting (`rm`, `Remove-Item`,
  `branch -d/-D`, `worktree remove`), the user's rule (2026-10-28). A longer list (`reset --hard`, `clean`, `checkout --`,
  `stash drop`, deploys) is the safer default if the user wants more protection, but every extra entry is a prompt. Ask
  rules beat allow rules, so a broad allow like `git checkout:*` can sit next to specific asks.
- **Worktrees:** a session started in a fresh worktree does not get the gitignored `settings.local.json`, so put
  `additionalDirectories` for the sibling worktrees (or their parent folder) in `~/.claude/settings.json`, and
  `defaultMode` in the committed `.claude/settings.json`; otherwise reads and edits across worktrees prompt.
- **Never allow** anything that runs arbitrary code: interpreters (`python`, `node -e`), shells, package runners, `make *`,
  `npm run *`, `gh api *`, and `git rebase` with a wildcard (`--exec` runs shell commands; allow only the exact forms).
  Interpreter wildcards also bypass the deny-read rules above, since code can read the files Claude may not.
- **`git -C <dir> ...` slips past prefix rules.** A rule like `git status:*` or `git push:*` matches only commands that
  start with it, so `git -C ../wt status` prompts (allow side) and `git -C ../wt push` is not asked about (ask side).
  Projects that work across worktrees add the mid-wildcard forms to `ask` (`git * push*`, `git * reset --hard*`, ...,
  as in the template). Check `/permissions` after editing: if the ask count is lower than the file has, a pattern was
  rejected; add rules in small groups to find it. A bare `Bash` allow is not a way to "allow everything but": the
  ask rules were not enforced in one session that had it, so keep explicit allows.
- **Most prompts come from compound commands, not the commands themselves:** `cd x && <write>` is checked as a write, and
  `python - <<EOF` patch scripts are code. Tell Claude to use Edit/Write for file changes. A `PreToolUse` Bash hook that
  blocks `sed -i` and heredoc patch scripts keeps Bash writes from skipping the edit guards and post-edit tests; do not
  block `cd`, it only matters when Bash is not allowed outright.
- **Turn a long build chain into one script** (e.g. `.claude/scripts/build.mjs`) with one exact allow rule. The script
  can hard-code the single path it deletes, which no wildcard rule can promise.
- To find what to allow, scan recent transcripts for the most frequent commands (the `fewer-permission-prompts` skill).

### Skills and connectors per project

Every skill's name and description, and every connector's instructions and some of its tools, are sent with every
request whether or not they're used. Turn off what a project doesn't need in its committed `.claude/settings.json`,
so it applies automatically in that project and nowhere else:

- **claude.ai skills:** `"skillOverrides": { "pptx": "off", ... }`, using the short names (the `name:` in each
  `~/.claude/skills/synced/*/<skill>/SKILL.md`). `"off"` hides it from Claude and the `/` menu;
  `"user-invocable-only"` hides it from Claude but keeps `/name`. Keep `handoff` and `release` on: the context hooks
  have Claude run `/handoff` itself. The `/skills` menu can write this too (Space to cycle, Esc to save). Plugin
  skills (superpowers) aren't affected; a plugin is only on or off as a whole, through `/plugin`.
- **claude.ai connectors:** `"disableClaudeAiConnectors": true` (the template default) drops all of them. Remove it
  in projects that use one; to block just one, list it in `deniedMcpServers` by name (e.g. `"claude.ai Slack"`).
  There's no "ask before turning on": MCP tool search already loads most tools only when Claude looks them up, and
  `/mcp` toggles a connector per project when the switch above isn't set.

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
| `ctx` green (under 150K tokens) | Plenty of room | Keep going |
| `ctx` yellow (150K-299K) + "/handoff, then /clear at the next stop" | Every message now re-sends a large conversation, and instruction-following gets worse | Finish the current step, `/handoff` (or `/release`), `/clear` |
| `ctx` red (300K or more) + "stop at the next safe point" | Expensive per message, and on a 200K-token model close to auto-compaction | Stop at the next safe point and do the same now |
| "cache cold" + "/handoff, then /clear" (context 60K or more) | The cache expired; the next message re-reads everything uncached | `/handoff`, `/clear`, start fresh rather than continue |
| `5h` red (80% or more) + "near the 5-hour limit" | The plan window is nearly used up | `/handoff` now, so nothing is lost if the limit hits mid-task |

Thresholds are in tokens, not percent: on a 1M-token window, "half full" is 500K tokens, long past the point where
a fresh chat is cheaper. The rule is about the conversation itself, which `/context` shows as **Messages**: plan a
break past about **60K of Messages**, stop past about **120K**. The total adds a fixed start-up cost (system prompt,
tools, skills, `CLAUDE.md`) that differs by surface:

| Surface | Fresh session starts at | Plan a break at (total) | Stop at (total) |
| --- | --- | --- | --- |
| Claude Code terminal, VS Code | about 35K | 95K | 155K |
| Claude app (desktop, web) | about 95K, mostly connector tools | about 155K | about 215K |

So in the app a 150K total is only about 55K of work; read the Messages line, not the total. Turning off unused
integrations (MCP connectors via `/mcp`, plugins via `enabledPlugins`) lowers the start-up cost. The lower thresholds
(they were 115K / 265K) came from a usage report: a third of a week's usage was spent past 150K of context.

You never need to run `/context` to calibrate anything. A hook or Claude can't run it (it is a built-in command whose
output goes to the screen), but every reply in the session log records its token total, and the start-up cost is the
smallest total in the log: the first reply, or the lowest point after a `/compact`. The context hooks measure it
that way on every prompt. The status line (terminal only) can't read the log, so it adds `CTX_BASELINE` (default 35000)
to the two budgets; set that variable to the total `/context` shows in a fresh chat if yours differs. Set it and the
hooks use it instead of the measured value.

### Context hooks

Two hooks share `.claude/hooks/context-usage.mjs`, which reads each reply's token total from the session log and
subtracts the start-up cost measured as above. A session with a single reply never warns. Both run locally, use no
tokens, and stay silent on any error.

- `context-warn.mjs` (UserPromptSubmit) shows you one line past 60K / 120K of conversation and gives Claude the same
  instruction: past 60K finish the current task, don't autorun further tasks, run the handoff routine, tell you to
  `/clear`; past 120K finish only the step in progress. It replaces running `/context` by hand.
- `context-stop.mjs` (Stop) fires when a turn ends past 60K with no handoff recorded this session, and blocks the stop
  once so Claude runs `/handoff` then. Waiting for the turn to end means the task finishes first. It fires once per
  session and never loops. A session where `/handoff` or `/release` already ran is skipped.

`/clear` itself can't be run by a hook or by Claude, so it stays your step.

## Working on several areas at once

The failure to design against: each branch edits its own copy of the handoff and ideas ledger, so a result recorded on an
unmerged branch is invisible to every other session, idea numbers collide, and the handoff on main describes a project
state that no longer exists.

- **One area = one branch = one worktree = one session.** Sessions never share a checkout.
- **Shared records live on main only.** The handoff (global state), the ideas ledger and the archive change in a
  permanent ledger worktree and land on main by fast-forward push. Hooks refuse the edit anywhere else.
- **Branch notes travel with the branch** (`docs/handoffs/<branch>.md`); nothing in them needs to be true on main.
- **State is read from git, not kept by hand.** A script lists every branch (ahead/behind, pushed, landed, worktree,
  Next line); a hand-kept table of branches rots the way the handoff did.
- **Claim a number by landing it.** A new idea row lands at once; a rejected push means the number was taken.
- The push is still the user's call: landing the ledger counts as a push, so it runs on "push" or /release only.

## End-of-session routine

1. Update the handoff: state, open items, "Next session starts with".
2. Update `IDEAS.md` rows for anything tested or shipped; move finished items and replaced decisions to the archive.
3. Check the size. If the handoff is over budget, archive the oldest material until it fits.
4. Change `CLAUDE.md` only when a rule, command or model-routing choice changed.
5. End the report with a **message to paste** into the fresh chat after `/clear` (first step, branch or worktree, what waits on
   the user, "run `branches.mjs`, then read the handoff"). The user clears after every handoff and should never have to
   re-explain. The rule lives in the project's "Release and handoff specifics", which `/handoff` reads.

The `/handoff` skill runs this routine the same way every time, without pushing.
