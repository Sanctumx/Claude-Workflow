# Working on this repo

Read `docs/SESSION_HANDOFF.md` first for the current state of the project. This file only covers how work gets done.
Every idea tried or planned has one line in `docs/IDEAS.md`; full write-ups are in `docs/HANDOFF_ARCHIVE.md`. Search those
(with the `explorer` subagent) when a task needs them; don't read them whole.

## Ideas and experiments

- Before proposing or testing an idea, have `explorer` search `docs/IDEAS.md` for related keywords. If it was Rejected,
  say so and re-test only if something changed (new data, different format, a bug in the old test).
- After a test or a shipped feature: edit its row in place (status and the deciding numbers), add the write-up to the
  archive, and update "Decisions in force" in the handoff if it was adopted.
- New ideas get the next number with status Open. Never delete rows.

## Standing rules

- Push, merge or deploy only when I say "push" (or run /release).
- Keep running plan tasks back to back unless something needs my input; but once a task has taken the conversation past
  the context limit (Messages past about 60K in `/context`), finish that task, record it, and do not autorun further
  tasks: tell me to `/handoff` and `/clear`.
- Scripts I run myself (e.g. Windows .bat files for long jobs) show only one updating "Estimated time remaining" line and
  close on their own when finished: quiet console, everything else to the results file, no `pause`.
- <Add your own: e.g. "never deploy to X unless asked", "report results in Y metric".>

## Commands

- Install: `<command>`
- Test: `<command>`
- Build: `<command>`
- Run locally: `<command>`

## Workflow

1. **Branch** from an up-to-date main: `git fetch origin && git checkout -b <short-name> origin/main`.
2. **Design first** for anything bigger than a small fix, sized as in "Superpowers skills": a medium task gets the
   planner's plan in the chat; a big feature gets a spec in `docs/specs/` (or wherever the brainstorming skill saves it),
   approved by me before building.
3. **Test first** where practical: write the failing test, watch it fail, make it pass.
4. **Commit** in small steps with a type prefix: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`.
5. **Review** before merging a feature: run the tests and linter, then the `reviewer` subagent reads the whole branch diff. Fix what matters, each fix with a test.
6. **Hand off** at the end of a session with `/handoff`: update `docs/SESSION_HANDOFF.md`, `docs/IDEAS.md` and the archive, keep the handoff under about 150 lines, and never push. Pushing is a separate, explicit step.

## Keeping context lean (`.claude/`)

- **Blocked from reading** (`settings.json` deny rules): secrets, dependencies, build output, minified files and lock
  files, plus <your large data, logs, result files and generated files>. Scripts still read them; to inspect one, run a
  short snippet that prints only what you need.
- **Path rules** (`.claude/rules/`): rules for one area load only when its files are opened.
- **Hook** (`.claude/hooks/after-edit.mjs`): after an edit to tested code it runs the unit tests and reports failures;
  after an edit to the handoff it warns past 150 lines. Fix what it reports before moving on.
- **Context hooks** (`.claude/hooks/context-warn.mjs`, `context-stop.mjs`): once the conversation passes about 60K
  tokens (stop past 120K) I get a one-line warning and you get the same instruction: finish the current task, don't
  start further plan tasks, run `/handoff`, then tell me to `/clear`. If a turn ends past 60K with no handoff
  recorded, the Stop hook makes you run it once. They work in the app's Claude Code tab, VS Code and the terminal, and
  measure the start-up cost from the session log (no need to run `/context`); `CTX_BASELINE` overrides it.
- **Status line** (`.claude/statusline.mjs`): model and effort, context %, 5-hour plan usage, cache state, branch,
  and a hint when it's time to `/handoff` and `/clear` (cues in `GUIDELINES.md`). It shows only in the terminal. In
  the Claude app or VS Code panel, the context hooks give the same warning; read **Messages** in `/context`, not the
  total, if you check by hand: the app starts around 95K before any work.

## Model routing

The main session runs on **Sonnet at medium effort**: in Claude Code from `.claude/settings.json`, in the Claude app by
keeping the model picker on Sonnet. Judgment work goes to subagents, each pinned to its own model and effort in
`.claude/agents/`, so the right model and effort are picked automatically and the main session never switches model
(its cache stays warm).

| Work | Who does it | Model · effort |
| --- | --- | --- |
| Building: UI, pages, plumbing, config, docs, carrying out plans | Main session | Sonnet · medium |
| Planning anything bigger than a small fix, architecture or data-model decisions, and any change in a hard area | `planner` subagent | Opus · medium (dispatch with effort high for a hard area) |
| Designing experiments and judging their results (delete `analyst.md` if the project has none) | `analyst` subagent | Opus · high |
| Searching the repo and docs (don't read large folders or data files in the main session) | `explorer` subagent | Haiku · low |
| Reviewing a finished branch (dispatch it with model sonnet if the branch only changes docs, config or wording) | `reviewer` subagent | Opus · medium (dispatch with effort high for a hard area) |

**How a planned task runs:** the main session (often with the explorer) gathers the task, the relevant files and any
related `IDEAS.md` rows, briefs the planner with all of it (the planner doesn't see this conversation), shows me the plan
it returns, and builds it once I approve. If the plan needs a decision, ask me before building.

**Escalation, only when needed:** a bug Sonnet has failed to fix twice, or a long stretch of edits inside a hard area,
may need Opus in the main session. Tell me, since only I can switch (`/model opus`, then `/model sonnet` after). For
one hard question, `ultrathink` in that prompt deepens one turn without changing settings.

**Hard areas** (subtle mistakes, no crash when wrong): <e.g. `src/engine.js` win-probability math, auth, payments, migrations>

## Superpowers skills

They work alongside the routing above; this file takes precedence over them. Size the process to the task:

| Task | Process |
| --- | --- |
| Small fix, UI tweak, copy or docs change | Just do it: test first where there's a test, and verify before saying done. No brainstorming, plan file or worktree. |
| Medium task (a few files, clear goal) | The `planner` subagent's short plan in the chat, approved by me; then test-first and verification. |
| Big feature, or anything that changes a hard area | Full flow: brainstorming (main session, it needs me), then the `planner` drafts the writing-plans document from that skill's format and the main session saves it, then executing-plans inline, then one review. |

- **Skill budget** (overrides using-superpowers' "invoke at 1% chance"): for small and medium tasks don't load
  test-driven-development or executing-plans; the test-first rule in this file is enough. Load them only for a big
  feature. Never run writing-plans or executing-plans as subagents.
- **Main model:** Sonnet unless I chose otherwise. The VS Code panel's and the app's model picker override
  `settings.json`, so if the session is on Opus and I haven't said it's deliberate, mention it once.
- **Skills and connectors:** unused claude.ai skills are turned off in `settings.json` (`skillOverrides`) and claude.ai
  connectors are off (`disableClaudeAiConnectors`; a `true` anywhere wins, so `/mcp` can't turn one back on). To use
  one, edit those lines in `.claude/settings.json` (Claude Code reloads it mid-session).
- **Planning:** writing-plans' thinking is done by the `planner` subagent (Opus); the main session only saves the file.
- **Executing:** use executing-plans inline by default. Use subagent-driven-development only for tasks that are truly
  independent of each other, and give every subagent an explicit model, as that skill requires.
- **Reviewing:** one review per branch. When a skill asks for a code review, run the `reviewer` subagent with that
  skill's template (Opus; Sonnet only for docs, config or wording-only branches).
- **Worktrees:** fine to use; the after-edit hook tests the checkout being edited.

## Release and handoff specifics (read by /release and /handoff)

- **Files:** `docs/SESSION_HANDOFF.md`, `docs/IDEAS.md`, `docs/HANDOFF_ARCHIVE.md`.
- **Results metric:** <how experiment results are reported, or "none">
- **Mirror the handoff:** <where a copy goes, e.g. a Claude project, or "none">
- **Handoff also tracks:** <versions or links worth keeping current, or "nothing extra">
- **Build check:** <what to build and test before a release, and when it's needed>
- **Deploy:** <what pushing or merging deploys, and anything never to do>
- **Never during /release or /handoff:** <e.g. "use my computer", or "nothing">

## Project map

- `<folder or file>`: <one line on what it is>
- `<folder or file>`: <one line>

## Gotchas

- <Things that went wrong before and how to avoid them. One line each.>
- On Windows, Bash heredocs with apostrophes in the text break: write such files with the Write tool instead.
