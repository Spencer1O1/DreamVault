## 1. Overview

Dream's long-term architecture should use **Gimbal as its formal semantic core**.

Dream remains the fuzzy, human-facing pseudocode language.

Gimbal provides the precise, typed, categorical program representation.

Target-language translation happens only after Dream's meaning has been formalized in Gimbal.

The long-term pipeline is:

```text
Dream Project
    ↓
.foo Semantic Units
    ↓
Dream Semantic Elaboration
    ↓
Gimbal Program
    ↓
Categorical Target Translation
    ↓
Target Project
    ↓
Builder
```

The defining relationship is:

> **Dream determines what the programmer meant. Gimbal determines what program that meaning represents exactly.**

---

# 2. Semantic Atom

The foundational long-term rule is:

> **One `.foo` file is one independently elaborated semantic unit.**

Dream does not attempt to discover semantic-unit boundaries inside an arbitrary pseudocode file.

The filesystem provides those boundaries deterministically.

Example:

```text
users/
├── load.foo
├── active.foo
├── oldest.foo
└── save.foo
```

Each file may eventually correspond to one formal Gimbal semantic fragment.

---

# 3. Why File-Level Units Matter

Dream source syntax is intentionally fuzzy.

Therefore something like:

```text
load the users

filter them down to active accounts

sort them oldest first
```

does not provide a deterministic parser-visible definition boundary.

Trying to detect internal units through the LLM would make caching and invalidation circular:

```text
need semantic units
↓
ask model where semantic units are
↓
cache semantics using boundaries produced by model
↓
source changes
↓
must ask model again to know whether boundaries changed
```

Dream avoids this entirely.

The programmer defines semantic boundaries through `.foo` files.

---

# 4. Structural Identity

Each semantic unit has deterministic structural identity:

```text
canonical project-relative path
```

Example:

```text
users/active.foo
```

That path identifies the unit regardless of how the model understands its contents.

This provides stable addressing for:

- semantic caches;
    
- locks;
    
- diagnostics;
    
- re-dreaming;
    
- dependency edges;
    
- source hashes;
    
- Gimbal fragments.
    

---

# 5. Source Graph

Dream imports define a deterministic graph before semantic elaboration.

Example:

```text
main.foo
├── users/load.foo
├── users/active.foo
│   └── models/user.foo
└── users/oldest.foo
```

This graph exists independently of the LLM.

It should be possible to construct and inspect it mechanically.

---

# 6. Semantic Graph

Eventually each source unit should elaborate into a corresponding semantic node.

Conceptually:

```text
main.foo
    ↓
Gimbal fragment M

users/load.foo
    ↓
Gimbal fragment L

users/active.foo
    ↓
Gimbal fragment A

users/oldest.foo
    ↓
Gimbal fragment O
```

The semantic dependency graph may be richer than the source import graph, but it should remain rooted in deterministic file-level unit identity.

---

# 7. Dream's Role

Dream is the fuzzy semantic frontend.

Its responsibility is:

```text
.foo pseudocode
    ↓
determine programmer intent
    ↓
produce valid Gimbal semantics
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
    

---

# 8. Gimbal's Role

Gimbal is the formal semantic core.

Its responsibility is:

```text
Dream semantic intent
    ↓
typed Gimbal program
    ↓
categorically structured semantics
```

Gimbal answers:

> **What program is this, exactly?**

Once a Dream unit has been accepted as Gimbal, its meaning no longer depends on fuzzy source interpretation during ordinary target translation.

---

# 9. Target Translation

Target translation should operate from Gimbal rather than directly from Dream source.

Conceptually:

```text
Gimbal Program
    ├── F_Rust
    ├── F_Go
    ├── F_C
    ├── F_Python
    └── ...
```

Each target translator maps the same formal program into another language ecosystem.

The exact mathematical formulation depends on Gimbal's final categorical semantics.

The design goal is that target translations preserve the relevant semantic structure rather than independently regenerate behavior through another LLM call.

---

# 10. Language Functors

Because Gimbal is categorically oriented, target-language translation should ideally be expressible through structure-preserving mappings.

Conceptually:

```text
      Gimbal Category
            │
            ├── F_Rust
            ├── F_Go
            ├── F_C
            └── F_...
```

Each `F_Target` preserves the program structure required by Gimbal's semantics while translating it into the conventions of the target language.

The exact categorical laws and structures preserved by these mappings must follow Gimbal's formal specification.

Do not assume more structure than Gimbal actually proves.

---

# 11. One Interpretation, Many Targets

Without Gimbal:

```text
Dream
 ├── LLM → Rust
 ├── LLM → Go
 └── LLM → Python
```

Each target can receive a subtly different semantic interpretation.

With Gimbal:

```text
Dream
   ↓
one accepted semantic interpretation
   ↓
Gimbal
 ├── Rust
 ├── Go
 └── Python
```

The target choice happens only after meaning is fixed.

This is one of the major reasons Gimbal should become Dream's semantic core.

---

# 12. Long-Term Default Pipeline

```text
Dream Project
     ↓
Source Resolver
     ↓
.foo Semantic Unit Graph
     ↓
Incremental Semantic Elaborator
     ↓
Gimbal Fragments
     ↓
Validated Gimbal Program
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

---

# 13. `dream now`

`dream now` remains intentionally different.

It can continue using the direct interpretation path:

```text
Dream Project
    ↓
Source Resolver
    ↓
LLM Interpreter
    ↓
Program Output
```

This keeps immediate Dream execution:

- permissive;
    
- fast to invoke;
    
- model-driven;
    
- useful for experimentation.
    

Normal Dream and `dream now` therefore serve distinct purposes.

---

# 14. Normal Dream

Long-term:

```bash
dream app.foo -t rust
```

should mean:

```text
Dream source
↓
incrementally elaborate required .foo units into Gimbal
↓
validate combined Gimbal program
↓
translate Gimbal to Rust
↓
create Rust project
↓
build
```

The LLM should not independently invent Rust semantics after the Gimbal program exists.

---

# 15. Semantic Persistence

Accepted Dream meaning should be persistent.

Core principle:

> **Dream source is fuzzy, but accepted meaning is stable until invalidated.**

Once:

```text
users/active.foo
```

has been elaborated into a valid Gimbal fragment, Dream should reuse that fragment unless something relevant changes.

---

# 16. Semantic Cache Entry

A semantic unit may eventually store metadata such as:

```text
canonical_path
source_hash
semantic_hash
Gimbal fragment
public semantic interface
semantic dependencies
frontend version
model/version
strictness
assumptions
lock state
```

Example:

```text
users/active.foo

source hash:
    92f...

semantic state:
    valid

Gimbal:
    <fragment>

interface:
    List<User> → List<User>

dependencies:
    models/user.foo

locked:
    true
```

---

# 17. Source Hashes

Each `.foo` file receives a deterministic source hash.

If the hash has not changed, Dream can know that the source text itself is unchanged without invoking the model.

This is the first level of cache validity.

---

# 18. Semantic Hashes

Accepted Gimbal meaning may also receive a semantic hash.

This allows Dream to determine whether a re-dreamed unit changed meaning even when its source changed.

Example:

```text
source hash changed
semantic hash unchanged
```

could mean the pseudocode was rewritten without changing the accepted program semantics.

---

# 19. Re-Dreaming

**Re-dreaming** is the process of semantically elaborating a `.foo` unit again.

Conceptual CLI:

```bash
dream redream users/oldest.foo
```

This means:

1. invalidate the accepted semantic result for that unit;
    
2. reinterpret the unit;
    
3. produce a new Gimbal fragment;
    
4. validate it;
    
5. compare its interface and semantic hash;
    
6. propagate invalidation only when necessary.
    

---

# 20. Incremental Re-Dreaming

Suppose:

```text
users/
├── active.foo
├── oldest.foo
└── save.foo
```

Only `oldest.foo` changes.

Desired behavior:

```text
active.foo → reuse cached Gimbal
oldest.foo → re-dream
save.foo   → reuse cached Gimbal
```

No unrelated LLM calls are required.

---

# 21. Dependency-Aware Invalidation

After re-dreaming a changed unit:

```text
new Gimbal fragment
    ↓
compare semantic interface
```

If the interface did not change:

```text
dependents may remain valid
```

If the interface changed:

```text
invalidate semantic dependents
```

Conceptually:

```text
source changed?
    ↓
re-dream unit
    ↓
public semantic interface changed?
    ├── no  → preserve dependents
    └── yes → invalidate dependents transitively
```

---

# 22. Example

Before:

```text
users/oldest.foo
    ↓
List<User> → List<User>
```

After editing:

```text
users/oldest.foo
    ↓
List<User> → List<User>
```

Implementation semantics may have changed from:

```text
sort by age
```

to:

```text
sort by signup date
```

but the structural type interface is unchanged.

A dependent unit that merely calls `oldest(users)` may not need semantic re-elaboration.

Its runtime behavior changes because its dependency changed, but its own accepted semantics remain valid.

---

# 23. More Than Types

Long-term invalidation may depend on more than type signatures.

A semantic interface may need to include:

- type;
    
- effects;
    
- capabilities;
    
- exported symbols;
    
- contracts;
    
- categorical structure;
    
- other Gimbal-visible behavior.
    

Exactly what constitutes a stable interface should be determined by Gimbal's semantics.

---

# 24. Locking

Dream should support locking accepted semantics.

Conceptually:

```bash
dream lock users/active.foo
```

A locked semantic unit means:

> Do not reinterpret this source unit automatically unless its lock is explicitly invalidated or a required semantic dependency makes the existing meaning unusable.

Exact lock behavior must remain compatible with correctness.

A lock should never force Dream to use an invalid Gimbal fragment.

---

# 25. Project Locking

Possible:

```bash
dream lock .
```

Meaning:

> Persist the currently accepted semantic representation of all resolved `.foo` units.

This could create a stable semantic snapshot of the project.

---

# 26. Semantic Lock Is Not Source Lock

The programmer can still edit:

```text
users/active.foo
```

A semantic lock records accepted meaning.

If source and lock diverge, Dream should clearly report the unit as stale rather than silently pretending they match.

---

# 27. Semantic States

Possible internal states:

```text
Missing
Valid
Locked
Stale
Invalid
```

Example:

```text
users/active.foo    Valid
users/oldest.foo    Stale
users/save.foo      Locked
```

Exact state design can evolve.

---

# 28. Inspecting Semantic State

Future:

```bash
dream inspect users/active.foo
```

could show:

```text
Unit:
  users/active.foo

Source:
  unchanged

Semantic state:
  locked

Interface:
  List<User> → List<User>

Dependencies:
  models/user.foo

Target-independent:
  yes
```

---

# 29. Project Inspection

Possible:

```bash
dream inspect .
```

Output could summarize:

```text
main.foo
  valid

users/load.foo
  valid

users/active.foo
  locked

users/oldest.foo
  stale
  source changed

dashboard/render.foo
  valid
  depends on users/oldest.foo
```

---

# 30. Semantic Graph Inspection

Possible future command:

```bash
dream inspect . --semantic-graph
```

Conceptual graph:

```text
main.foo
├── users/load.foo
├── users/active.foo
└── users/oldest.foo
    └── models/user.foo
```

with metadata showing semantic status.

---

# 31. Gimbal Fragments

Each independently elaborated Dream unit should eventually correspond to a composable Gimbal fragment.

Conceptually:

```text
users/active.foo
    ↓
Gimbal fragment A
```

```text
users/oldest.foo
    ↓
Gimbal fragment B
```

```text
main.foo
    ↓
Gimbal fragment M
```

The project semantic phase composes these fragments into a valid whole.

---

# 32. Gimbal as the Semantic Lock Representation

Once Gimbal is sufficiently expressive, Dream should avoid inventing a separate semantic IR.

The accepted Gimbal representation itself can serve as Dream's locked meaning.

Instead of:

```text
Dream AST
↓
Dream IR
↓
Gimbal
```

prefer:

```text
Dream source
↓
semantic elaboration
↓
Gimbal
```

unless implementation experience proves an intermediate layer is necessary.

---

# 33. Strict Mode

Gimbal gives strict mode a much stronger eventual definition.

Instead of merely:

> Tell the model to guess less.

strict mode can become:

> **The Dream frontend must produce a valid Gimbal semantic unit without unresolved assumptions forbidden by strict policy.**

Conceptually:

```text
.foo source
    ↓
semantic elaboration
    ↓
valid Gimbal fragment?
    ├── yes → accept
    └── no  → DreamError
```

---

# 34. Normal Mode

Normal mode may allow reasonable assumptions.

Those assumptions should eventually be recorded in semantic metadata.

Example:

```text
"sort users"
    ↓
assumed ordering: name ascending
```

If accepted, that assumption becomes part of the semantic result rather than being re-guessed every compilation.

---

# 35. Strict Mode and Locks

A semantic fragment accepted under normal mode may not automatically satisfy strict mode.

Cache keys should therefore include semantic policy such as:

```text
strictness
frontend version
```

A normal-mode semantic result should not be silently reused as a strict-mode result unless Dream can formally establish that it satisfies strict requirements.

---

# 36. Semantic Frontend Versioning

Dream's semantic behavior depends on its elaboration prompt, model interaction protocol, and policies.

Long-term semantic cache entries should record:

```text
frontend_version
model
semantic_policy
```

This allows Dream to detect when a previously accepted interpretation came from an incompatible frontend.

---

# 37. Model Drift

Model behavior can change.

Persistent Gimbal semantics protect already accepted code from unnecessary model drift.

Once a unit has:

```text
valid Gimbal meaning
```

target generation no longer needs to reinterpret that source.

This greatly reduces dependence on model consistency.

---

# 38. Language as a Service

Dream's incremental semantic architecture makes a hosted Dream service economically plausible.

Instead of sending an entire project for reinterpretation every time, a client could provide:

```text
unit paths
source hashes
semantic hashes
dependency metadata
changed units
```

The service only re-dreams invalid semantic units.

---

# 39. Hosted Semantic Elaboration

Conceptually:

```text
Local Dream CLI
    ↓
changed .foo units
    ↓
Dream Language Service
    ↓
semantic elaboration
    ↓
Gimbal fragments
    ↓
local cache / lock
```

Target translation may then happen locally or remotely depending on product design.

---

# 40. Incremental Service Model

Example project has 2,000 Dream units.

One file changes:

```text
payments/calculate_tax.foo
```

Desired hosted behavior:

```text
1 changed source unit
↓
1 semantic cache miss
↓
re-dream that unit
↓
validate its interface
↓
invalidate only affected dependents
```

Not:

```text
send entire project to model again
```

This is essential for scalability.

---

# 41. Content Addressing

Semantic caching should eventually be content-addressable where useful.

A cache key might include:

```text
source hash
dependency interface hashes
strictness
frontend version
semantic environment
```

The exact key should be designed around correctness rather than merely token savings.

---

# 42. Dependency Interface Hashes

A unit may depend on the interfaces of imported units rather than their complete implementation semantics.

If dependency implementation changes but its exported interface hash remains the same, dependent semantic results may remain valid.

Conceptually:

```text
semantic cache key:
    own source hash
    +
    dependency interface hashes
    +
    semantic frontend version
```

This is similar in spirit to incremental compilation.

---

# 43. Dream Becomes an Incremental Semantic Compiler Frontend

Long-term, the LLM should not behave like:

```text
the compiler that reruns on everything
```

It should behave more like:

> **An incremental semantic elaborator that runs only where program meaning is missing or stale.**

That is a much stronger architecture.

---

# 44. Target Independence

Once a `.foo` unit has valid Gimbal semantics, that semantic result should generally be target-independent.

This is crucial.

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
Gimbal meaning
    ├── Rust
    ├── Go
    └── Python
```

---

# 45. Target Translation Must Not Re-Dream

Once Gimbal exists, translating to another target should not semantically reinterpret Dream source.

Example:

```bash
dream app.foo -t rust
dream app.foo -t go
```

should reuse the same accepted Gimbal semantics.

Only the target translation differs.

---

# 46. Target Functor Cache

Eventually target translation itself can also be cached separately.

Conceptually:

```text
Gimbal semantic hash
    +
target translator version
    +
target configuration
```

This cache is deterministic and distinct from Dream semantic caching.

---

# 47. Layered Caching

Long-term cache layers:

```text
Dream source
    ↓
Semantic Cache
    ↓
Gimbal
    ↓
Target Translation Cache
    ↓
Target Project
    ↓
Build Cache
    ↓
Artifact
```

Each layer has different invalidation rules.

---

# 48. Source Changes

A source edit should ideally invalidate only:

```text
changed semantic unit
+
semantic dependents whose required interface changed
+
target artifacts derived from changed semantics
```

Nothing more.

---

# 49. Target Changes

Changing only:

```bash
-t rust
```

to:

```bash
-t go
```

should not invalidate Dream semantics.

It should only require a new target translation.

---

# 50. Build Configuration Changes

Changing:

```text
debug → release
```

should ideally not require:

- re-dreaming;
    
- Gimbal regeneration;
    
- target source regeneration unless required.
    

Only the Builder stage may need to rerun.

---

# 51. Layer Responsibility

## Source Resolver

Owns:

- files;
    
- imports;
    
- project graph.
    

## Dream Semantic Frontend

Owns:

- pseudocode interpretation;
    
- ambiguity resolution;
    
- Gimbal elaboration.
    

## Gimbal

Owns:

- formal program representation;
    
- typing;
    
- semantic validity;
    
- categorical structure.
    

## Target Translator

Owns:

- Gimbal → target semantics/source.
    

## Builder

Owns:

- target toolchain execution.
    

## Runner

Owns:

- program process execution.
    

---

# 52. Final Long-Term Architecture

```text
                         Dream Project
                              │
                              ▼
                       Source Resolver
                              │
                              ▼
                 Deterministic .foo Unit Graph
                              │
                              ▼
                 Semantic Cache / Lock Lookup
                              │
                 ┌────────────┴────────────┐
                 │                         │
             cache hit                 stale/missing
                 │                         │
                 │                         ▼
                 │                 Dream Semantic
                 │                   Elaborator
                 │                         │
                 └────────────┬────────────┘
                              ▼
                       Gimbal Fragments
                              │
                              ▼
                    Validated Gimbal Program
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
              F_Rust        F_Go       F_Python
                 │            │            │
                 ▼            ▼            ▼
           Rust Project   Go Project   Python Project
                 │            │            │
                 ▼            ▼            ▼
              Builder      Builder      Builder
```

---

# 53. Long-Term `dream now`

```text
Dream Project
    ↓
Source Resolver
    ↓
Resolved .foo Graph
    ↓
Interpreter
    ↓
Program Output
```

`dream now` may eventually reuse accepted semantic metadata where useful, but it remains conceptually the immediate interpretation path.

---

# 54. Long-Term Normal Dream

```bash
dream app.foo -t rust
```

eventually means:

```text
resolve project
↓
determine stale semantic units
↓
re-dream only those units
↓
compose validated Gimbal program
↓
translate Gimbal through Rust target semantics
↓
write Rust project
↓
build
```

---

# 55. Core Long-Term Principles

## Semantic Atom

> **One `.foo` file is one semantic unit.**

## Structural Determinism

> **Dream's project graph is determined mechanically, not by the LLM.**

## Incremental Semantics

> **Only stale or invalid semantic units should be re-dreamed.**

## Persistent Meaning

> **Once a Dream unit has accepted Gimbal semantics, that meaning persists until invalidated.**

## Interface-Aware Invalidation

> **Dependents should only be semantically invalidated when relevant dependency interfaces change.**

## Formal Core

> **Gimbal is Dream's long-term semantic representation.**

## Target Independence

> **Dream semantics are fixed before target selection is applied.**

## Functorial Translation

> **Target mappings should preserve the relevant categorical structure of Gimbal.**

## Capability Ownership

> **Dream owns machine capabilities. Models only perform semantic interpretation.**

---

# 56. Final Relationship Between Dream and Gimbal

Dream and Gimbal should eventually form two layers of the same language system.

```text
Dream
    =
informal human-facing notation

Gimbal
    =
formal categorical program semantics
```

Dream asks:

> **What did the programmer mean?**

Gimbal answers:

> **What program is that meaning, exactly?**

Target translators answer:

> **How does this formal program live in another language ecosystem?**

---

# 57. Final Vision

```text
Human Intent
    ↓
Dream
    ↓
Executable Pseudocode
    ↓
Incremental Semantic Elaboration
    ↓
Persistent Gimbal Meaning
    ↓
Categorical Translation
    ├── Rust
    ├── Go
    ├── C
    ├── Python
    └── ...
```

The long-term goal is not to repeatedly ask an AI to rewrite a program.

The goal is:

> **Use AI only where human pseudocode is ambiguous, persist the resulting formal meaning, and let deterministic semantics take over from there.**

That is what makes Dream viable as a real programming language rather than merely an LLM wrapper.