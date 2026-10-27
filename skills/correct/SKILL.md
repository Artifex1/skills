---
name: correct
description: Make a repeating mistake impossible, fixing each class /qa reports under Repeats at the highest level that works.
disable-model-invocation: true
---

# Correct

The agents in this repo see only the files they open, copy the nearest example, and take the shortest path that compiles. When `/qa` reports **Repeats**, change the repo so the next agent cannot make that mistake again. Adapted from pstack's `correct`.

The **findings log**, **check** and **targeted test** roles are bound in `docs/agents/workflow.md`. If that file is missing, set it up with the user from [`ship`'s WORKFLOW.md](../ship/WORKFLOW.md) first.

## 1. Pick the classes

Read the findings log and take every class with two or more entries and no `corrected` line. Read the code behind its entries until you can describe the mistake's shape in one sentence.

## 2. Fix each at the highest level that works

1. **Architecture**: one owner per piece of state, one supported way per task; hide internals so the wrong import fails to compile; delete the old way an agent would copy.
2. **Types**: make the bad state unwritable.
3. **A check whose error names the fix**: a test that scans the source for the pattern, or a linter rule. Fix or explicitly allow the existing occurrences.
4. **A behaviour test.**
5. **AGENTS.md or docs**, last, and only for judgement calls: nothing fails when an agent skips them.

Done for a class when its level is chosen and you can say why each higher level doesn't work.

## 3. Prove

Every new check must fail on a real past mistake from the log: reintroduce it, watch the check fail with an error that names the fix, revert. Run only the checks you added, as targeted tests. Ask once for permission to commit, then commit one class per commit.

## 4. Mark

Append to the findings log:

```
<date> | <class> | corrected | <level> | <check path or commit>
```

## Report

Each class: its entry count, the level chosen and why higher levels didn't work, and the proof that its check fails on the past mistake.
