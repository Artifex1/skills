# Workflow

The binding between the ticket-flow skills (`to-tickets`, `ship`, `qa`, `correct`, `grill`, `improve-codebase-architecture`) and one repo. The skills name **roles**; this file says where each role lives here and which commands run. Copy it to `docs/agents/workflow.md` and replace every example with this repo's answer.

## Tracker

- **Ticket**: where one ticket lives and what its id is.
  _Local:_ `.scratch/<feature>/issues/<NN>-<slug>.md`, id `<NN>`, one folder per feature. _Jira:_ one issue per ticket in project `KEY`, one epic per feature, through the Atlassian MCP tools.
- **Blocking**: how a ticket names the tickets that gate it.
  _Local:_ a `Blocked by:` line. _Jira:_ "is blocked by" links.
- **Status**: how a ticket's status is recorded, and the value for each role: `ready-for-agent` (fully specified), `ready-for-human` (needs the user), `done`.
  _Local:_ a `Status:` line. _Jira:_ the workflow states.
- **Decision log**: one per `ship` run, status `ready-for-human`, one line per entry, sortable.
  _Local:_ `.scratch/<feature>/decisions.md`. _Notion:_ a page per run under a "Decisions" page.
- **Findings log**: append-only, one line per confirmed QA finding, readable in full so classes can be counted.
  _Local:_ `.scratch/qa/findings.md`. _Notion:_ a database with columns date, class, severity, location, outcome.

## Docs

- **Map**: the one-page architecture map. Keep it in the repo, next to the code: the rule that a change updates it in the same commit, and the user's review of its diff, both depend on that.
  _Default:_ `docs/architecture.md`.
- **Glossary**: _default_ `GLOSSARY.md`.
- **ADRs**: _default_ `docs/adr/`.

## Commands

- **Check**: the cheapest command that sees the whole change set. _e.g._ `cargo check`, `tsc --noEmit`.
- **Targeted test**: tests of one area. _e.g._ `cargo test -p <crate> <filter>`, `vitest run <path>`.
- **Full suite**: _e.g._ `cargo test --workspace`, `pnpm test`.

## Git

- **Integration**: where a run's commits go by default, and whether a run ends in a PR.
  _e.g._ the current branch, no PR; or a branch `ship/<feature>` per run, pushed, with a PR opened at the end (`gh pr create`).

## Agents

- **Implementer**: model and effort for `ship`'s implementers and the fixers. _e.g._ `sonnet`, or `opus` at `medium`.
- **Reviewer**: model and effort for `qa`'s lenses and the decision panel. _e.g._ `opus`.
