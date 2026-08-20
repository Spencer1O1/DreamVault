**Status:** Deferred  
**Purpose:** Capture a candidate long-term formal semantic core. This is not part of the current Dream design. Gimbal has not been figured out yet.

---

## Status

The current Dream architecture goes from resolved `.foo` units to an Interpreter or Composer.

A later architecture may introduce a precise, typed program representation between Dream meaning and target-language translation.

[[Gimbal]] is one candidate for that representation.

Nothing below is a current commitment.

The pre-Gimbal pipeline remains:

```text
Dream units
    ↓
Composer (0..N target artifacts per unit)
    ↓
Target Project
```

Locks, before this core exists, freeze those artifacts. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

See [[Projects/Dream/Architecture|Architecture]] and [[Projects/Dream/Implementation Plan|Implementation Plan]].

## Candidate Relationship

If a formal core is adopted, the intended relationship would be:

> **Dream determines what the programmer meant. The formal core determines what program that meaning represents exactly.**

Dream would remain the fuzzy, human-facing pseudocode language.

The formal core would provide the precise program representation.

Target-language translation would happen only after Dream's meaning had been formalized.

## Candidate Pipeline

```text
Dream Project
    ↓
.foo Semantic Units
    ↓
Dream Semantic Elaboration
    ↓
Formal Program
    ↓
Target Translation
    ↓
Target Project
    ↓
Builder
```

## Why a Formal Core Might Matter

Without a shared formal interpretation:

```text
Dream
 ├── LLM → Rust
 ├── LLM → Go
 └── LLM → Python
```

Each target can receive a subtly different semantic interpretation.

With a formal core:

```text
Dream
   ↓
one accepted semantic interpretation
   ↓
formal program
 ├── Rust
 ├── Go
 └── Python
```

The target choice happens only after meaning is fixed.

That is the main reason a formal core is attractive.

It is also the reason this should wait until that core actually exists.

## Dream's Role in That Architecture

Dream would remain the fuzzy semantic frontend.

Its responsibility would be:

```text
.foo pseudocode
    ↓
determine programmer intent
    ↓
produce valid formal semantics
```

Dream answers:

> **What did the programmer mean?**

It handles:

- informal syntax;
- ordinary pseudocode;
- natural programming phrasing;
- local ambiguity;
- programmer intent;
- semantic inference.

## Formal Core's Role

The formal core would answer:

> **What program is this, exactly?**

Once a Dream unit had been accepted as a formal fragment, its meaning would no longer depend on fuzzy source interpretation during ordinary target translation.

## Target Independence

Once a `.foo` unit has valid formal semantics, that semantic result should generally be target-independent.

Bad:

```text
users/active.foo
    ↓
Rust meaning

users/active.foo
    ↓
Go meaning
```

Desired:

```text
users/active.foo
    ↓
formal meaning
    ├── Rust
    ├── Go
    └── Python
```

Translating to another target should not semantically reinterpret Dream source.

Example:

```bash
dream app.foo -t rust
dream app.foo -t go
```

should reuse the same accepted formal semantics.

Only the target translation differs.

## Semantic Graph

Eventually each source unit could elaborate into a corresponding semantic node.

Conceptually:

```text
main.foo
    ↓
formal fragment M

users/load.foo
    ↓
formal fragment L

users/active.foo
    ↓
formal fragment A

users/oldest.foo
    ↓
formal fragment O
```

The semantic dependency graph may be richer than the recorded source-request graph, but it should remain rooted in deterministic file-level unit identity.

## Avoid Inventing a Separate IR Too Early

If a formal core becomes expressive enough, Dream should avoid inventing a separate semantic IR just to sit in front of it.

Instead of:

```text
Dream AST
↓
Dream IR
↓
formal core
```

prefer:

```text
Dream source
↓
semantic elaboration
↓
formal core
```

unless implementation experience proves an intermediate layer is necessary.

## Long-Term Default Pipeline, If Adopted

```text
Dream Project
     ↓
Source Resolver
     ↓
.foo Semantic Unit Graph
     ↓
Incremental Semantic Elaborator
     ↓
formal fragments
     ↓
validated formal program
     ↓
Target-Language Translation
     ↓
Target Project
     ↓
Builder
     ↓
Built Artifact
     ↓
optional Runner
```

`dream now` can remain the direct interpretation path even in that architecture.

## Layer Responsibility in That Architecture

### Source Resolver

Owns:

- files;
- path sandbox;
- recorded source-request graph.

### Dream Semantic Frontend

Owns:

- pseudocode interpretation;
- ambiguity resolution;
- formal elaboration.

### Formal Core

Owns:

- formal program representation;
- typing;
- semantic validity.

### Target Translator

Owns:

- formal program → target semantics/source.

### Builder

Owns:

- target toolchain execution.

### Runner

Owns:

- program process execution.

## Gimbal as a Candidate

Dream's long-term notes originally named [[Gimbal]] as the formal semantic core.

That remains a possible later choice.

It is not a current design input.

Gimbal would need to be mature enough that each `.foo` semantic unit can elaborate into a corresponding Gimbal fragment or declaration.

If Gimbal is used, target translation should operate from Gimbal rather than directly from Dream source.

Because Gimbal is categorically oriented, those translations might later be expressible as structure-preserving mappings.

Do not assume more structure than Gimbal actually proves.

Do not design Dream's current MVP around that possibility.

## Related

- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]]
- [[Projects/Dream/Implementation Plan|Implementation Plan]]
- [[Projects/Dream/Vision and Principles|Vision and Principles]]
- [[Projects/Dream/Later Interpreter Runtime|Later Interpreter Runtime]]
