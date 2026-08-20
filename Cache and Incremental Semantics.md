**Status:** Later, after the first composition pipeline  
**Purpose:** Define file-level semantic caching, re-dreaming, and interface-aware invalidation.

---

## Source Hashing

Each semantic unit should eventually have a source hash.

Conceptually:

```text
UnitKey:
    canonical_path
    source_hash
```

This allows Dream to detect whether the source for that semantic unit changed.

No LLM is needed to answer that question.

If the hash has not changed, Dream can know that the source text itself is unchanged without invoking the model.

This is the first level of cache validity.

## Semantic Cache

Long-term, each `.foo` unit may have cached semantic metadata.

Conceptually:

```text
users/active.foo

source_hash
semantic_hash
formal_representation
public_interface
dependencies
frontend_version
strictness
assumptions
```

The cache should be attached to the file-level semantic unit.

A cache key might eventually include:

```text
source hash
dependency interface hashes
strictness
frontend version
semantic environment
```

The exact key should be designed around correctness rather than merely token savings.

## Semantic Persistence

Accepted Dream meaning should be persistent.

Core principle:

> **Dream source is fuzzy, but accepted meaning is stable until invalidated.**

Once a unit has been elaborated into an accepted semantic result, Dream should reuse that result unless something relevant changes.

## Semantic Hashes

Accepted meaning may also receive a semantic hash.

This allows Dream to determine whether a re-dreamed unit changed meaning even when its source changed.

Example:

```text
source hash changed
semantic hash unchanged
```

could mean the pseudocode was rewritten without changing the accepted program semantics.

## Re-Dreaming

**Re-dreaming** means recomputing the semantic meaning of a specific `.foo` unit.

Conceptually:

```bash
dream redream users/active.foo
```

This should invalidate that semantic unit and derive its meaning again.

Because `.foo` files are semantic atoms, re-dreaming has a deterministic boundary.

## Incremental Semantics

The long-term architecture should support incremental semantic invalidation.

Example:

```text
users/
├── active.foo
└── oldest.foo
```

If only:

```text
oldest.foo
```

changes, Dream should ideally reuse the accepted meaning of:

```text
active.foo
```

and only re-dream the changed unit.

No unrelated LLM calls are required.

## Interface-Aware Invalidation

Suppose:

```text
oldest.foo
```

previously represented:

```text
List<User> → List<User>
```

and after re-dreaming still represents:

```text
List<User> → List<User>
```

Then dependents may not need semantic reinterpretation.

Conceptually:

```text
source changed?
    ↓ yes
re-dream unit
    ↓
semantic interface changed?
    ├── no  → dependents remain valid
    └── yes → invalidate dependents
```

This is the desired long-term incremental model.

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

## More Than Types

Long-term invalidation may depend on more than type signatures.

A semantic interface may need to include:

- type;
- effects;
- capabilities;
- exported symbols;
- contracts;
- other visible behavior.

Exactly what constitutes a stable interface depends on the later formal semantic representation. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

## Dependency Interface Hashes

A unit may depend on the interfaces of the `.foo` files it requested rather than their complete implementation semantics.

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

## Semantic Frontend Versioning

Dream's semantic behavior depends on its elaboration prompt, model interaction protocol, and policies.

Long-term semantic cache entries should record:

```text
frontend_version
model
semantic_policy
```

This allows Dream to detect when a previously accepted interpretation came from an incompatible frontend.

## Model Drift

Model behavior can change.

Persistent accepted semantics protect already accepted code from unnecessary model drift.

Once a unit has valid accepted meaning, later work on that unit should not need to reinterpret that source unless the unit is stale or invalid.

## Dream Becomes an Incremental Semantic Frontend

Long-term, the LLM should not behave like:

```text
the compiler that reruns on everything
```

It should behave more like:

> **An incremental semantic elaborator that runs only where program meaning is missing or stale.**

That is a much stronger architecture.

## Layered Caching

Long-term cache layers:

```text
Dream source
    ↓
Semantic Cache
    ↓
accepted meaning
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

## Source Changes

A source edit should ideally invalidate only:

```text
changed semantic unit
+
semantic dependents whose required interface changed
+
target artifacts derived from changed semantics
```

Nothing more.

## Target Changes

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

## Build Configuration Changes

Changing:

```text
debug → release
```

should ideally not require:

- re-dreaming;
- semantic regeneration;
- target source regeneration unless required.

Only the Builder stage may need to rerun.

## Language as a Service

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

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Semantic Locking and Inspection|Semantic Locking and Inspection]]
- [[Projects/Dream/Semantics and Strictness|Semantics and Strictness]]
- [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]]
