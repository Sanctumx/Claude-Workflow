# Claude workflow templates

Starting files for Claude Code projects that use several models efficiently: Opus subagents for planning and judgment,
Sonnet for building, Haiku for searching, plus a short handoff and an ideas ledger so new chats start cheap.

## Use in a new project

**Quickest:** run `/new-project` in the project (a skill on the Claude account). It copies `template/`, asks a few
questions, fills everything in and checks it. The manual steps below do the same by hand.

1. Copy everything inside `template/` into the new project's root, including the hidden `.claude/` folder.
2. Start Claude Code there and say: *"Read GUIDELINES.md, then fill in CLAUDE.md, docs/SESSION_HANDOFF.md and
   docs/IDEAS.md for this project. Ask me what you need to know."*
3. On GitHub, in the new repo's Settings → General → Pull Requests, tick **Automatically delete head branches**.
   It is a per-repo setting, and Claude sessions can't delete branches or change repo settings themselves.
4. After the first working version: *"Update CLAUDE.md with the real commands, project map and release specifics, set
   the deny rules and the hook's test settings."*

## What's in `template/`

| Path | What it is |
| --- | --- |
| `GUIDELINES.md` | The rules behind everything else: what goes where, size budgets, when to release, compact or clear |
| `CLAUDE.md` | How to work in the project: rules, commands, workflow, model routing, release specifics |
| `docs/SESSION_HANDOFF.md` | Current state only, about 150 lines max |
| `docs/IDEAS.md` | One row per idea tried, adopted, rejected or open |
| `docs/HANDOFF_ARCHIVE.md` | Full write-ups, read only when needed |
| `.claude/settings.json` | Main session on Sonnet at medium effort, deny-read rules, the after-edit hook, the status line |
| `.claude/agents/` | `planner` (Opus), `analyst` (Opus), `reviewer` (Opus), `explorer` (Haiku), each with its own effort |
| `.claude/rules/hard-area.md` | Template for rules that load only when one area's files are opened |
| `.claude/hooks/after-edit.mjs` | Runs fast unit tests after edits; warns when the handoff grows too long |
| `.claude/hooks/context-warn.mjs` | Warns you before a prompt is sent when the conversation passes 115K / 265K tokens, and tells Claude to finish the task and hand off |
| `.claude/hooks/context-stop.mjs` | Stop hook: past 115K with no handoff yet, makes Claude run `/handoff` once when its turn ends |
| `.claude/hooks/context-usage.mjs` | Shared token-count reader and thresholds for the two context hooks |
| `.claude/statusline.mjs` | Status line: context, plan usage, cache, branch, and when to `/clear` |

The `/release` and `/handoff` skills live on the Claude account; they read each project's "Release and handoff
specifics" section in `CLAUDE.md`.

## Keeping this current

Every workflow improvement made in a project is also generalized here. Change the template, update `GUIDELINES.md` if
the rule behind it changed, and commit.
