---
name: grill
description: A relentless interview to sharpen a plan or design, recording ADRs, glossary terms and map changes as decisions settle.
disable-model-invocation: true
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round, each question with your recommended answer. Then wait for the user's answers before the next round.

Ask through the harness's question tool when it has one (`AskUserQuestion` in Claude Code, or its equivalent): one question per frontier decision, its choices as options, your recommendation first and marked "(Recommended)", the context the user needs in the question itself. The tool takes a few questions per call, so a larger frontier goes out in consecutive calls, most foundational first. The user can always answer outside the options.

Without a question tool, write the round as text, like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

As decisions settle, record them, using the Skill tool with "domain-modeling" for the formats: a new or sharpened term goes into the glossary; a decision a future reader needs goes into an ADR (offer it, don't impose it); a decision that changes the system's outside shape updates the map. The glossary, ADRs and map are bound in `docs/agents/workflow.md`. If that file is missing, set it up with the user from [`ship`'s WORKFLOW.md](../ship/WORKFLOW.md) first.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding; then offer `/to-tickets`.
