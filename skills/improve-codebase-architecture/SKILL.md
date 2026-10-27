---
name: improve-codebase-architecture
description: Scan the codebase for deepening opportunities, present them as candidates in chat, then grill or ticket whichever one you pick.
disable-model-invocation: true
---
# Improve Codebase Architecture

Surface architectural friction and propose **deepening opportunities**: refactors that turn shallow modules into deep ones. The aim is testability and AI-navigability. Where `/qa` hunts defects in recent changes, this hunts the structure that keeps producing them.

## Vocabulary

Use these terms exactly in every suggestion; don't drift into "component," "service," "API," or "boundary."

- **Module**: anything with an interface and an implementation, at any scale: a function, a type, a crate.
- **Interface**: everything a caller must know to use the module: the signature, plus invariants, ordering, error modes, configuration and performance.
- **Depth**: behaviour a caller can exercise per unit of interface it must learn. **Deep** = a lot of behaviour behind a small interface; **shallow** = an interface nearly as complex as the implementation.
- **Seam**: the place a module's interface lives, where behaviour can change without editing in that place.
- **Adapter**: a concrete thing that fills a seam. One adapter means a hypothetical seam; two mean a real one.
- **Leverage**: what callers get from depth. **Locality**: what maintainers get: change, bugs and verification concentrated in one place.
- **Deletion test**: imagine deleting the module. If complexity vanishes, it was a pass-through; if it reappears across callers, it earns its keep.
- **The interface is the test surface**: wanting to test past the interface means the module is the wrong shape.

The domain language in the glossary names good seams; the ADRs record decisions this skill doesn't re-litigate; the map shows the parts and connections. The glossary, ADRs, map and findings log are bound in `docs/agents/workflow.md`. If that file is missing, set it up with the user from [`ship`'s WORKFLOW.md](../ship/WORKFLOW.md) first.

## Process

### 1. Explore

**Scope before you scan: YAGNI.** Deepening a module pays off by making future changes to it easier, so weight the parts of the codebase that change most.

- If the user named a direction (a module, a subsystem, a pain point), take it.
- Otherwise walk back a good stretch of `git log --oneline` (and the findings log, if it has entries) to find the hot spots, and let those paths pull your attention first.

Read the glossary, the map, and the ADRs in the area first. Then spawn a sub-agent to walk the code. Explore organically and note where you experience friction:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules **shallow**?
- Where have pure functions been extracted just for testability, while the real bugs hide in how they're called (no **locality**)?
- Where do tightly-coupled modules leak across their seams?
- Which parts are untested, or hard to test through their current interface?

Apply the deletion test to anything you suspect is shallow.

### 2. Present candidates

In chat, one short card per candidate:

- **Files**: the modules involved
- **Problem**: the friction the current shape causes
- **Solution**: in plain English, what would change
- **Benefits**: in terms of locality and leverage, and how tests would improve
- **Strength**: `Strong`, `Worth exploring` or `Speculative`

Name things in the glossary's vocabulary ("the Order intake module", not "the FooHandler"). Surface a candidate that contradicts an ADR only when the friction is real enough to reopen it, and say so on the card ("contradicts ADR 0007, worth reopening because…").

End with a **top recommendation**: which candidate to tackle first and why. Don't propose interfaces yet; ask which candidate the user wants to pursue.

### 3. Pursue the pick

- **Shape still open**: run `/grill` on it: constraints, dependencies, the shape of the deepened module, what sits behind the seam, which tests survive.
- **Shape clear**: go straight to `/to-tickets`, prefactoring first.

When the user rejects a candidate for a load-bearing reason, offer an ADR so future reviews don't re-suggest it; skip ephemeral reasons ("not now").
