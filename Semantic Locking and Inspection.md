**Status:** Later (Phase 10). Needs provenance (Phase 8).  
**Purpose:** Persist accepted target realizations for `.foo` units and make that state inspectable.

There is **no** `redream` command. Normal `dream` is the reconcile. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

---

## Pre-Gimbal Locks

Before Gimbal, a lock is **target-specific**.

```bash
dream lock server.foo -t rust
```

Means: preserve the currently accepted artifact set **and contents** for that unit in that target.

```text
server.foo [locked for rust]
  → src/server.rs
  → src/routes.rs
```

Normal `dream` must not let the Composer mutate those paths.

Source-oriented: the user locks the `.foo` file. Dream already knows the generated set.

Do not invent target-independent locked meaning before Gimbal.

`read_source_file` still returns the `.foo`. In compose mode, if Dream has a realization (locked, or settled / just composed this run), the result includes those artifact paths and contents. An unlocked unit that is not settled yet is composed first (nested job), then returned the same way. The interpreter (`dream now`) does not attach artifacts and does not recurse into compose.

See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

---

## Unlock

```bash
dream unlock server.foo -t rust
```

Then a normal `dream` may recompose that unit.

`--fresh` ignores target-specific locks.

If the locked unit’s **source hash** no longer matches the hash stored at lock time: `DreamError`. Unlock or restore the `.foo`.

If a locked artifact path is **missing**: `DreamError`. Restore it, unlock, or `--fresh`. Do not compose it from foocode.

If a locked artifact was **edited** by hand: leave it. Dream still will not overwrite it.

---

## File Requests Do Not Block Locking

A lock records the *result* of a compose, not a parsed import list.

After `server.foo` is composed, Dream has:

```text
artifact set
contents
requested .foo files
source hash (when that exists)
```

Locking freezes the artifacts. Informal file requests are part of how the unit was dreamed; the recorded graph can stay with the unit.

If a requested dependency later becomes unusable (missing file, broken interface), the lock is stale. A lock must never force an invalid result.

---

## Semantic Lock Is Not Source Lock

The programmer can still edit `server.foo`.

If source and lock diverge, Dream should report the unit as stale rather than silently pretending they match. First lock CLI does not have to implement inspect.

---

## States

Possible:

```text
Missing
Valid
Locked
Stale
Invalid
```

Exact state design can evolve. Do not persist a speculative enum before lock CLI exists.

---

## Inspecting (Later)

```bash
dream inspect server.foo
```

could show path, source unchanged?, lock, artifact set, dependencies.

`dream inspect .` could summarize the project.

Not required for Phase 10.

---

## After Gimbal

A lock should freeze formal meaning. Provenance still tracks the translated files. Changing `-t` should not require re-elaboration.

That is later. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

---

## Related

- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
