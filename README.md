# Skills

A lean skill set for engineering with agents: iterate fast while keeping the code good. You shape the design and write tickets, implementer agents build them one per ticket to a finished-design standard, and a stronger reviewer model runs QA every few tickets for bugs, debt, performance and test quality. An architecture map and a decision log keep every structural change visible and reviewable.

The skills are plain `SKILL.md` folders and assume nothing about the harness beyond subagents and, where available, a tool for asking the user structured questions. Model names in them are examples: a cheaper implementer and a stronger reviewer, in whatever family your harness runs.

## Install

With the GitHub CLI or the `skills` CLI, which place them where your agent loads skills from:

```sh
gh skill install Artifex1/skills --all   # pick the agent with --agent, user-wide with --scope user
npx skills add Artifex1/skills
```

Or copy the folders under [`skills/`](skills/) yourself, for one project or for all of them (for example `.claude/skills/` or `~/.claude/skills/` for Claude Code, `~/.codex/skills/` for Codex). The ticket-flow skills link to each other by relative path, so install the set together.

The ticket-flow skills name roles (ticket, decision log, findings log, map, commands, models) instead of paths. On first use they set up `docs/agents/workflow.md` with you from [`skills/ship/WORKFLOW.md`](skills/ship/WORKFLOW.md), which binds each role to the repo: local markdown, Jira or Notion for tickets and logs, your check and test commands, your implementer and reviewer models.

## Which skill when

| When you want to… | Use | Starts |
|---|---|---|
| Sharpen a plan or design before building it | `/grill` | you |
| Turn a plan, spec or conversation into tickets | `/to-tickets` | you |
| Build a batch of tickets | `/ship` | you |
| Review what was built: bugs, debt, performance, hollow tests | `/qa` | you |
| Stop a mistake QA keeps finding (when `/qa` reports **Repeats**) | `/correct` | you |
| Find structural friction worth refactoring | `/improve-codebase-architecture` | you |
| Work at pace: change in bulk, test where it has leverage | `/chopchop` | you |
| Have the last message re-pitched in plain words | `/wait-what` | you |
| Debug a hard bug or performance regression | `diagnosing-bugs` | agent or you |
| Record glossary terms and ADRs | `domain-modeling` | agent or you |
| Research a question against primary sources | `research` | agent or you |
| Write or edit a skill or an agent instruction file | `writing-for-agents` | agent or you |

The usual loop: `/grill` → `/to-tickets` → `/ship` → `/qa` → `/correct` when repeats show up, and `/improve-codebase-architecture` now and then. Skills marked "you" set `disable-model-invocation: true`: they start only when typed, so their descriptions cost the agent no context until used. A harness that ignores that field lets the agent reach them too.

## Where they come from

- **[Matt Pocock's skills](https://github.com/mattpocock/skills)** (v1.3.1): `diagnosing-bugs`, `domain-modeling`, `research`, `wait-what` and `writing-for-agents` largely as published; `to-tickets` and `improve-codebase-architecture` adapted (tracker roles, a **Verify by:** line per ticket, candidates in chat instead of an HTML report); `grill` merges his `grilling`, `grill-me` and `grill-with-docs`.
- **[pstack](https://github.com/cursor/plugins/tree/main/pstack)** (Cursor): `correct` adapts its `correct`, fixing a repeated mistake at the highest level that works (architecture, types, a check, a test, docs last). `qa` takes the rule from its `blast-radius`: prove the one fact a finding rests on by running real code.
- **[tstack](https://github.com/t3dotgg/tstack)** (Theo Browne): `qa`'s severity scale, round budget and resistance to scope creep come from `sol-loop` and `council`; `ship`'s rule to ask the user only when it cannot continue, through the question tool, comes from `full-send`.
- **Lydia from the Claude newsletter**: `chopchop` is built around "Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better."
- **Anthropic's model guidance** ([choosing a model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model), [cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)): the implementer and reviewer models are settings to measure rather than fixed choices, and tuning effort is often a better lever than switching models.
