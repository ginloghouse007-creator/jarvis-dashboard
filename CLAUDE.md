# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

Static, no-build dashboard: `index.html`, `jarvis-toolbox.html`, `data.json`.
No package manager, no framework, no server. Keep it that way unless asked
to change it — don't introduce a build step or framework for its own sake.

## Available subagents and commands

A small, curated set imported from [ECC](https://github.com/affaan-m/ECC)
(MIT; see `.claude/THIRD_PARTY_NOTICES.md`) lives in `.claude/agents/` and
`.claude/commands/`:

- `/plan` — restate requirements, surface risks, get a step plan approved
  before touching code. Use for anything non-trivial.
- `/code-review` — review local uncommitted changes (or a PR).
- Subagents (invoke via the Agent tool when a task calls for a focused pass):
  `planner`, `architect`, `code-reviewer`, `security-reviewer`,
  `silent-failure-hunter`, `refactor-cleaner`.

These are on-demand only — they add no cost unless invoked. Do not treat
their presence as license to over-engineer this project; `refactor-cleaner`'s
npm-ecosystem tooling (knip/depcheck/ts-prune) won't apply here since there's
no `package.json` — skip those steps gracefully if run.

## Security checklist for this project (client-side, static site)

- This is a static site with a real XSS surface: `index.html` and
  `jarvis-toolbox.html` write untrusted/dynamic strings into the DOM via
  `innerHTML` in several places (e.g. `jarvis-toolbox.html`'s task-input
  tool builds `li.innerHTML` directly from user-typed text). Prefer
  `textContent` / DOM node creation over `innerHTML` for any value that
  isn't a fixed, hardcoded string. Run `security-reviewer` before adding
  new `innerHTML` usage.
- Never hardcode secrets, API keys, or tokens in these files — they are
  served as static assets and are effectively public.
- Keep `data.json` as trusted, self-authored data. If it ever starts being
  written by an external source, revisit the XSS points above first.

## Git workflow

- Commit messages: `<type>: <description>` (`feat`, `fix`, `refactor`,
  `docs`, `test`, `chore`).
- Before committing: no `console.log`/debug leftovers, no secrets.

## Review bar (from ECC's code-review rule, trimmed to this project's scale)

- Functions stay focused; avoid deep (>4 level) nesting.
- Handle errors explicitly — no silently swallowed failures.
- CRITICAL (security/data loss) issues block; HIGH issues should be fixed
  before considering a change done; LOW/style notes are optional.
