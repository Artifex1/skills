---
name: qa
description: QA pass over a commit range, hunting bugs, code quality, layered debt, performance and hollow tests; fresh fixers fix what is confirmed.
disable-model-invocation: true
argument-hint: <commit range, e.g. main~8..main>
---

# QA

This pass is the quality gate for everything the implementers build. Hunt real defects in a commit range, prove them, fix them, and log them so repeating mistakes get corrected at the root.

The roles below (**ticket**, **map**, **findings log**, **check**, **targeted test**, **full suite**, **integration**, **implementer**, **reviewer**) are bound in `docs/agents/workflow.md`. If that file is missing, set it up with the user from [`ship`'s WORKFLOW.md](../ship/WORKFLOW.md) first.

## 1. Scope

Pin the range: the argument, else everything since the last commit whose message starts with `qa:`. Read `git log --oneline <range>`, `git diff --stat <range>`, and the tickets the commits name. Done when you can state the range's intent in two sentences.

Ask once, now, where the fixes this pass makes are committed (see **integration**), and for permission to commit them there.

## 2. Hunt

Spawn one read-only reviewer agent per lens, in parallel. Each gets the range, the tickets, its lens, and the severity scale.

- **Correctness**: bugs, the ticket's intent missed, error and edge paths, concurrency, data loss. Look past the diff: callers, wire formats, on-disk data and public surfaces the change also moves.
- **Quality and performance**: the code must read as if designed with this requirement from the start. Flag the *layers* a patch leaves behind: shims, flags for an old path, the old way kept beside the new, duplicate paths, TODOs, workarounds where a restructure was due. Flag the opposite too: speculative generality, abstractions and options nothing needs. Also duplication, shallow modules, dead code, needless work or allocation on hot paths, and blocking calls on a UI thread or async runtime.
- **Tests**: each new or changed test must fail when its behaviour breaks. Flag *tautological* tests (expected value computed the way the code computes it), *implementation-coupled* tests (break on a refactor, survive a behaviour change), and tests that assert nothing that matters. A hollow test is deleted, not patched: fewer, real tests.
- **Map**: does the map still describe the system after this range? A structural change missing from it is a finding.

Severity: **blocker** = the intended case is wrong; **should** = a real bug, or debt a future change will pay for; **nit** = everything else, including other paths, rare cases and small costs.

Finding format: `[severity] file:line: claim. Evidence: <file:line chain, failing command, or script output>. Fix: <the proper fix>`. When a finding rests on one fact that cheap code can settle, run the real code and paste the output.

## 3. Confirm

Check every finding yourself against the code. Drop what doesn't hold; downgrade anything mislabelled. Done when each finding is confirmed with evidence or dropped.

## 4. Fix

Blockers and shoulds go to fresh implementer agents, one area of the code each, one after another in the checkout under review. Brief them with the implementer rules of `/ship`, copied verbatim from its "Per ticket" section ([SKILL.md](../ship/SKILL.md)); a fixer cannot load `/ship` itself, so the rules travel in the brief. The fix is the proper one, held to the finding's scope: restructure what the finding touches rather than patch around it, and nothing beyond. A fix that needs groundwork beyond the finding becomes a ticket instead. Nits are not fixed; they become follow-ups in the report.

Then the full suite once; one fixer for its failures. A second review round runs only while a blocker is open, and only over the code that changed. Two rounds at most. Commit as `qa: <range>: <summary>`.

## 5. Log

Append each confirmed finding to the findings log:

```
<date> | <class> | <severity> | <file:line> | fixed|deleted|ticketed|follow-up
```

A **class** is a short reusable name for the kind of mistake (`tautological-test`, `old-path-kept`, `blocking-io-on-ui-thread`). Reuse a class already in the log before coining one; a class is only useful if it recurs under one name.

## Report

The range, findings by severity with their outcomes, tests deleted, tickets written, the suite result, follow-ups, and a final line:

**Repeats:** each class now at two or more entries in the findings log and not yet corrected, which is the cue for `/correct`; or `none`.
