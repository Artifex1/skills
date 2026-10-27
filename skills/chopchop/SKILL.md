---
name: chopchop
description: Speed mode. Converge on a quality result in few, meaningful turns; change in bulk, test where it has leverage.
disable-model-invocation: true
---

# Chopchop

> Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better.

Quality comes first: the goal is a correct, well-made result. Speed comes from **convergence**, not from cutting corners or shaving seconds. Every turn moves the task measurably closer to done; a turn that only nudges, re-checks or re-reads what is already known is the time to avoid.

## Understand once

Read everything the change touches before the first edit: the call sites, the types, the tests that pin the behaviour. Done when you can list every edit the change needs, file by file.

## Change in bulk

Make the whole change set in one pass: every file, every call site, then verify. A rename or signature change lands everywhere at once, and the compiler tells you what is left.

## Test with leverage

A test run is a measurement: run one when its result can change what you do next.

1. **Check first.** The type checker (`cargo check`, `tsc`) is the cheapest run that sees the whole change set. Fix everything it reports in one batch, then rerun.
2. **Scope to the blast radius.** Run the tests of what you changed and its direct dependents: the touched crate (`cargo test -p <crate>`), a test-name filter. Code you did not touch and that was green is still green.
3. **One full suite, once, at the end**, and only when the change crosses modules or touches a shared interface.

After a failing run, read the whole output and fix every failure before the next run: a batch of fixes per run. Rerun only what failed.

When the same failure survives two fixes, stop guessing and diagnose (`diagnosing-bugs`).

## Report

State what you ran and what you deliberately skipped, so the user knows exactly what is covered.
