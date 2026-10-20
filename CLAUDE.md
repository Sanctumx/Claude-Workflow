# Working on this repo

This repo holds the general version of the user's Claude Code workflow. Everything a new project copies lives in
`template/`; `template/CLAUDE.md` is a template full of placeholders, not instructions for this repo.

## The user's rules

- Push only when the user says "push".
- Every workflow improvement made in a project is generalized here where possible (user, 2026-10-20).

## Generalizing an improvement from a project

1. Take the project's version (a rule, a subagent, a setting, a hook, a skill change) and remove what is specific to
   that project: names, paths, metrics, commands. Put placeholders `<like this>` where the project fills in its own.
2. Put it in the matching file under `template/`. If it changes a rule or the reasoning behind one, update
   `template/GUIDELINES.md` too, in the same plain style (short sentences, no project names).
3. If it adds a file, add a row to the table in `README.md`.
4. Keep every template file valid as it stands: `template/.claude/settings.json` must parse as JSON, and the hook must
   run with empty input and with a sample edit (`echo '{"tool_input":{"file_path":"docs/SESSION_HANDOFF.md"}}' | node
   template/.claude/hooks/after-edit.mjs` from inside `template/`, with `CLAUDE_PROJECT_DIR` set to it).
5. Commit with a type prefix (`feat:`, `fix:`, `docs:`, `chore:`) and a line naming the project it came from.

## Skills

`/release` and `/handoff` live on the user's Claude account, not here. They read each project's "Release and handoff
specifics" section in its `CLAUDE.md`. A change to how they work is proposed as a skill update, and the matching section
in `template/CLAUDE.md` is updated here.
