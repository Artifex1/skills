---
name: inbox
description: Put every open `yours` decision from /dispatch runs to the user, apply the answers to the tickets, and name the questions grill keeps missing.
disable-model-invocation: true
argument-hint: "[feature or ticket ids]"
---

# Inbox

`/dispatch` never waits for the user: when a decision is theirs and they are away, it takes the recommended option, leaves it open on the ticket as `- [ ] yours: …`, and carries on. This skill puts those decisions to the user.

The **ticket**, **open decision**, **status**, **reviewer**, **glossary** and **ADRs** roles are bound in `docs/agents/workflow.md`. If that file is missing, set it up with the user from [`dispatch`'s WORKFLOW.md](../dispatch/WORKFLOW.md) first.

## 1. Collect

Find every ticket with an open decision (the argument narrows it to a feature or tickets), and take each unchecked `- [ ] yours:` line from its `## Decisions` section. If there are none, say so and stop.

Hand the lines to one read-only reviewer agent (the configured model and effort; split by feature when there are many), so the reading stays out of this thread. Its brief is the lines with their tickets, and these rules, verbatim:

> - **Read** each ticket and the code its choice landed in, including later tickets that build on it.
> - **Plain words.** For each line, say what the user sees or what depends on it, without file paths unless a decision turns on one.
> - **Reversal.** Say what undoing it would take today: files, data, interfaces, later tickets. Rate it `cheap` or `expensive`.
> - **Recommend.** Re-judge from what you read which option you'd pick and why; don't copy the run's choice.
> - **Read-only.** Change no file and no git state.
> - **Report**, per line: the ticket, the decision as a question, what changes for the user, the real options, your recommendation with its reason, and the reversal with its rating.

## 2. Ask

Ask through the question tool, one question per decision, most expensive to reverse first. The tool takes a few questions per call, so a longer inbox goes out in consecutive calls.

- **Question**: the decision in plain words, what changes for the user, the ticket, and what reversing it costs today, from the agent's report.
- **Options**: keep the chosen option, and reverse to each real alternative. Mark the recommendation first with "(Recommended)".

Without a question tool, write the same as a numbered list, each with your recommendation, and wait for the answers.

## 3. Apply

- **Keep**: tick the line and append `kept`.
- **Reverse**: write a ticket for the reversal (the `/to-tickets` template, with a **Verify by:** line, status `ready-for-agent`, blocked by nothing it doesn't need), then tick the line and append `reversed → <new ticket id>`.
- **Another answer**: treat it as a reverse to what the user said.

When a ticket has no open line left, clear its open-decision marker. When an answer is one a future reader would wonder about, offer an ADR (use the Skill tool with "domain-modeling" for the format); don't impose it.

## 4. Report

Once: the decisions kept, the reversal tickets written (run them with `/dispatch`), and the ADRs added.

Then the gaps. Every `yours` decision is a question `/grill` didn't ask before the tickets were cut. Group this inbox's decisions by what they were about (error behaviour, naming users see, a data format); a group of two or more is a branch the next `/grill` should cover, so name it.
