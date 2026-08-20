**Status:** Foundational  
**Purpose:** Collect the invariants that should constrain Dream's design.

---

## Semantic Unit Rule

> **One `.foo` file is one semantic unit.**

## Source Structure Rule

> **Dream never depends on an LLM to discover semantic boundaries inside a `.foo` file.**

## Multi-File Rule

> **A `.foo` unit may use other `.foo` units. The model requests those files. Dream records the resulting graph.**

## Composition Rule

> **The Composer realizes `.foo` units. Dream's project layer owns infrastructure. Dream execs the declared toolchain.**

## Target Rule

> **Target generation is open-ended. Toolchain support is optional.**

Declare the toolchain before output writes. Do not infer it from the tree.

## Capability Rule

> **Dream owns capabilities. The model owns interpretation.**

The model asks through tools. Dream performs the effect. Chat text is not program output.

Source tools, interpreter runtime tools, and composer output tools are separate families. Do not mix them.

## Strictness Rule

> **`--strict` is a stricter prompt. It is not a parser.**

## Artifact Rule

> **Generated target projects are first-class outputs and remain useful without Dream.**

## Multi-Artifact Rule

> **One semantic unit may own zero, one, or many target artifacts.**

Do not require one `.foo` file to map to one generated file.

## Provenance Rule

> **Every Composer-generated artifact is owned by exactly one `.foo` unit.**

## Project Artifact Rule

> **Dream owns project infrastructure. The Composer does not edit those files directly.**

## Unmanaged Artifact Rule

> **Unknown files are user-owned. Normal composition must not delete or overwrite them.**

## Reconciliation Rule

> **Normal `dream` reconciles in place. It does not clear `-o`.**

There is no `redream` command. `--fresh` is the destructive regenerate.

## Pre-Gimbal Lock Rule

> **Before Gimbal, locking a `.foo` unit freezes its current target-specific artifact set.**

After Gimbal, locks freeze target-independent formal meaning. Do not fake that IR now.

## Incremental Semantics

> **Only unlocked units that need reconciliation should be recomposed.**

Normal `dream` is that operation.

## Persistent Meaning

> **Once a Dream unit has accepted meaning, that meaning persists until invalidated.**

Before Gimbal, accepted meaning for a target is that unit's artifact set and contents.

## Interface-Aware Invalidation

> **Dependents should only be semantically invalidated when relevant dependency interfaces change.**

## Structural Determinism

> **Unit identity and project boundaries are mechanical. Which files a unit uses is discovered, then persisted.**

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
- [[Projects/Dream/Vision and Principles|Vision and Principles]]
