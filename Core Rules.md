**Status:** Foundational  
**Purpose:** Collect the invariants that should constrain Dream's design.

---

## Semantic Unit Rule

> **One `.foo` file is one semantic unit.**

## Source Structure Rule

> **Dream never depends on an LLM to discover semantic boundaries inside a `.foo` file.**

## Multi-File Rule

> **Imports define the deterministic dependency graph between semantic units.**

## Composition Rule

> **The Composer creates target projects. The Builder builds them.**

## Target Rule

> **Target generation is open-ended. Builder support is optional.**

## Capability Rule

> **Dream owns capabilities. The model owns interpretation.**

## Strictness Rule

> **Normal Dream resolves reasonable ambiguity. Strict Dream exposes important ambiguity.**

## Artifact Rule

> **Generated target projects are first-class outputs and remain useful without Dream.**

## Incremental Semantics

> **Only stale or invalid semantic units should be re-dreamed.**

## Persistent Meaning

> **Once a Dream unit has accepted meaning, that meaning persists until invalidated.**

## Interface-Aware Invalidation

> **Dependents should only be semantically invalidated when relevant dependency interfaces change.**

## Structural Determinism

> **Dream's project graph is determined mechanically, not by the LLM.**

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
- [[Projects/Dream/Vision and Principles|Vision and Principles]]
