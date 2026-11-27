---
name: dispatch
description: Run a batch of tickets through fresh implementer agents, one per ticket in a shared checkout, recording every departure as a decision on its ticket.
disable-model-invocation: true
argument-hint: <feature or ticket ids>
---

# Dispatch

Implement a batch of tickets with implementer agents while the main thread only coordinates. Work at the `/chopchop` pace: quality first, every turn moving the batch toward done.

The roles below (**ticket**, **status**, **open decision**, **map**, **check**, **targeted test**, **full suite**, **integration**, **implementer**, **reviewer**) are bound in `docs/agents/workflow.md`. If that file is missing, set it up with the user from [WORKFLOW.md](WORKFLOW.md) before anything else.

## Start

1. Read the tickets, order them so blockers come first, and read the map.
2. Ask the user once, before the first agent, recommending the configured **integration**: where the run's commits go (the current branch, or a new branch for this batch), whether the run ends by pushing and opening a PR, and whether they are around for decisions or away. That answer covers every git step of the run.

## Per ticket

Spawn a fresh implementer agent (the configured model and effort) in the run's checkout. One ticket per agent, so every implementer starts on a clean context; what it needs from earlier tickets is in the commits and the tickets.

By default the agents run one at a time: each builds on the previous commit, so nothing ever merges. Run tickets in parallel only when neither blocks the other and their file sets are disjoint (judge it from the tickets and the map before spawning): they share the checkout, each runs only its own targeted tests and is told the other tickets in flight (an error from their files is not its own), and the coordinator commits each ticket's own files. Worktrees cost a fresh build each and merge back later, so use them only when the user asks. The user can steer any of this at the start.

The brief is the ticket plus these rules, verbatim, with the role names resolved to this repo's commands and paths:

> - **Finished design.** The code you leave reads as if the design had always included this requirement: restructure what the change touches, delete the old way, and leave no shims, flags for the old path, duplicate paths or TODOs. Effort and time are not constraints; the proper design is always the one to build. Proper means as few pieces as the requirement needs: no abstraction, option or layer the ticket doesn't call for. Your scope is what the ticket touches; debt you see beyond it goes in your report.
> - **Too big for the ticket.** When the proper design needs more than this ticket (a groundwork refactor, a change across many callers), stop and report what the groundwork is. The coordinator turns it into its own ticket.
> - **Tests.** For a bug, first write a test that fails on the bug, then fix it. For new behaviour, test where the ticket's **Verify by:** says, through the public interface. Expected values come from the ticket or a worked example, never recomputed the way the code computes them.
> - **Test runs.** The check, then only the targeted tests you wrote or directly touched. The full suite runs once after the batch.
> - **Departures.** When the work needs something other than what the ticket, an ADR, or the code being ported implies (including layout and structure when porting), stop and report the choice with its options, and the one you'd pick, instead of making it.
> - **Map.** When your change makes the map wrong, update it in the same change.
> - **Git.** Leave every git state change to the coordinator.
> - **Report:** what changed, tests added and run, each map edit with a one-line reason, departures, groundwork needed, debt seen beyond the scope, anything left undone.

Then the main thread:

- reads the report, not the diff;
- settles a departure (see Decisions) and sends a fresh agent to finish the ticket with the answer;
- for groundwork, writes a new ticket that blocks this one, ships it first, then sends a fresh agent back to this ticket;
- writes the ticket's decision lines and, under `## Debt seen`, the debt the implementer saw beyond its scope (the next `/qa` picks it up), sets its status to `done` (an open `yours` line doesn't hold it back), and commits `<ticket id>: <title>`, with each map edit and its reason in the message body (and the ticket file, when tickets live in the repo).

An agent that needed far more turns than its siblings had an oversized ticket: note it for the report.

## Decisions

Every departure gets a decision, and the decision is written on its ticket, in a `## Decisions` section, one line each:

```
- [x] panel: <chosen> over <alternative>, because <reason>; reverse by <how>.
- [ ] yours: <chosen> over <alternative>, because <reason>; reverse by <how>.
```

Sort each departure before settling it:

- **yours**: it changes what users see or do, what is in or out of scope, or something outside the repo depends on (an external or public interface, a stored data format, a security or trust boundary), or it contradicts an ADR.
- **panel**: everything else. The code and documents can answer it.

A `yours` decision never blocks the run. With the user around, ask through the question tool, recommendation first, and write the line as `- [x] asked:`. With the user away, take the recommended option (the most reversible one when it is close), write it as an open `- [ ] yours:` line, and carry on, including with the tickets that build on it. The open box is what `/inbox` collects later; mark the ticket the way the **open decision** role says. The run stops only for an action outside the repo (a migration against a shared database, publishing, sending): set that ticket to `ready-for-human`, skip the tickets it blocks, and continue with the rest.

A `panel` decision goes to the panel whether or not the user is around. Write the question neutrally: the decision and what it touches, with no options and no recommendation. Spawn two reviewer agents in parallel, each with the question, its reading list, and these rules, verbatim:

> - **Answer** the question in one or two sentences, from your reading list alone.
> - **Cite** what forces the answer: the file and line, quoted. A preference without a citation is not an answer.
> - **Unforced.** If nothing on your list forces an answer, reply `unforced` with what you checked.
> - **Read-only.** Change no file and no git state.

- **Documents**: the ticket, the spec, the ADRs and the glossary.
- **Code**: the map, the code the decision touches and its callers, and the places that already solve a similar problem.

Match each answer to the reported options yourself; an answer outside them is a real option. The leading option is the one the cited answers support. Two cited answers that disagree mean nothing forces it. Then send the question, the leading option and both answers, with these rules, to the outside reviewer when the workflow lists a second provider (the other provider's reviewer, through its headless command), else to a third reviewer agent:

> - **Attack.** Find the strongest case against the leading option: a concrete input, caller, or later ticket in this batch where it is wrong. Read anything you need. If you find none, say so; don't invent one.
> - **Citations.** Check that each quoted line says what its answer claims.
> - **Reversal.** Say what undoing the option would take: files, data, interfaces callers depend on. Rate it `cheap` (one module, no data, no public interface) or `expensive`.
> - **Read-only.** Change no file and no git state.
> - **Report:** the case or `none found`, each citation that doesn't hold, and the reversal with its rating.

Agents of one model agree by default, so agreement alone settles nothing:

- If a citation forces the answer, holds up, and the attack finds no expensive case, take it: `- [x] panel:`.
- If nothing forces it, take the most reversible option: `- [x] panel?:`.
- If nothing forces it and reversing would be expensive, take the most reversible option anyway, but leave it open for the user: `- [ ] yours:`.

Choices inside a module are not departures and get no line; `/qa` covers those.

## Finish

1. Run the full suite once in the background and read only the failures. Hand all of them to one fresh implementer agent, then rerun once.
2. If the user is around and any `yours` line is open, settle them now as [`inbox`](../inbox/SKILL.md) does: a reviewer agent collects them for this run's tickets, the main thread asks and applies the answers.
3. If the run ends in a PR: push the branch and open it, with the tickets done, a **Waiting on you** section listing every open `yours` line with its ticket, and the suite result in its description.
4. Report once, opening with **Waiting on you**: the `yours` lines still open with their tickets (answer them with `/inbox`) and tickets set to `ready-for-human` with the tickets they block. Then tickets done with their commits, the PR link, the suite result, groundwork tickets added, oversized tickets, and the commit range for `/qa`.

While agents run, stay silent: speak only when blocked or when a decision needs the user.
