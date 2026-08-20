**Status:** Current compose contract through Phase 9. Phase 10 (locks) is still next.  
**Purpose:** Pre-Gimbal target artifact ownership, in-place reconciliation, and locks. Implementers should follow this plus [[Projects/Dream/Core Rules|Core Rules]]. Do not invent a target-independent IR.

[[Projects/Dream/MVP|MVP]] is the historical v0 (replace `-o`, builder after writes). The crate no longer does that.

---

## Why This Exists

A generic “convert these `.foo` files” agent treats `-o` as disposable.

Dream cannot, once people add files, lock units, or run `dream` twice.

The accepted realization of a `.foo` unit, before Gimbal, is **that unit’s current target-specific artifact set**. Not a fake universal meaning.

---

## Source Rule Unchanged

> **One `.foo` file is one semantic unit.**

That unit may compose into **zero, one, or many** target artifacts.

```text
server.foo
  → cmd/server/main.go
  → internal/server/server.go
  → internal/server/routes.go
```

Do **not** require one `.foo` → one target file.

The invariant is **provenance**, not file-count.

---

## Ownership Classes

Every path under `-o` is one of:

```text
1. Unit-owned     — Composer realization of one .foo unit
2. Project-owned  — Dream’s target project layer (manifests, glue)
3. Unmanaged      — no Dream provenance; user-owned by default
```

Toolchain outputs (`target/`, `Cargo.lock`, …) are build state, not semantic artifacts. Do not treat them as Composer output. With no provenance store, any file outside `.dream/` means `-o` is occupied (`--fresh` or an empty directory). Do not skip leftover toolchain dirs by guessing names.

```text
owner = Unit(path)
owner = Project
owner = Unmanaged   // absent from the provenance store
```

Do not infer ownership from path shape. Use provenance.

---

## Provenance Map

Dream must answer mechanically, per target and `-o`:

- which artifacts came from this `.foo` unit?
- which unit owns this generated path?
- which paths may be replaced or removed?
- what does a lock freeze?
- what is unmanaged and therefore untouchable?

Conceptually:

```text
target: rust
output: ./out

units:
  server.foo:
    artifacts:
      - src/server.rs
      - src/routes.rs
    locked: false

project:
  - Cargo.toml
```

Storage format is not locked. Do not invent a speculative schema before the first persist.

A unit may also own generated resources (`assets/generated-background.svg`). Manually added `assets/logo.png` with no provenance stays unmanaged.

---

## Builder First

`-t` stays an open-ended compose hint.

**Declare the builder before any output writes.** That turn is `set_builder` only — no `dream_error`, no write tools. v0 asked after the write loop; that is gone.

```text
entry .foo
    ↓
set_builder (once)
    ↓
known builder → Dream may init / load project-owned state
unsupported  → compose still allowed; no --build / --run / repair
    ↓
reconcile units
    ↓
reconcile project-owned state
    ↓
--build / --run if asked
```

Do not infer the builder from the tree. Do not take argv from the model.

Known builders get structured project tools. Unknown / `unsupported` targets use a generic write fallback. **Ownership still applies.** Do not give the Composer unrestricted filesystem or shell access to support a weird `-t`.

---

## Composer Write Boundary

The Composer does not get unrestricted write access to `-o`.

A write names the `.foo` unit it belongs to. It may create, update, or delete only artifacts owned by that unit (including newly registered paths for it).

Reject:

- overwrite of another unit’s artifact;
- overwrite of a project-owned file (use a project tool);
- overwrite of an unmanaged file (unless a later explicit user grant exists).

The Composer must not silently steal ownership.

After the session settles, Dream records the new path set for each unit that wrote. Paths that were owned by that unit and are gone from the new set may be deleted.

---

## Deletion

Dream may delete a path on a normal run only when:

1. provenance proves Dream owns it (unit or project); and
2. that owner is allowed to mutate it (not locked); and
3. the path is no longer in that owner’s desired set.

Do not delete unrelated files. Do not clear `-o`.

---

## Normal `dream` Reconciles

There is **no** `redream` command.

```bash
dream app.foo -t rust -o ./out
```

means: reconcile this source graph into the **existing** target project, respecting locks and ownership.

```text
declare builder
    ↓
load existing target provenance
    ↓
one composition session (entry is the root)
    ↓
reconcile project-owned state
    ↓
build if asked
```

Normal `dream` must **not** `rm -rf` `-o`.

### Composition session

`dream <entry.foo>` starts **one** conversation. The entry is the discovery root, not the owner of every write. `.foo` remains the semantic, lock, and provenance unit. It is not the LLM invocation unit.

`read_source_file` never starts a compose job. It returns the foocode immediately, plus that unit’s artifacts if the provenance store already has them (last accepted realization, locked or not). `dream now` returns `{ path, source }` only.

A `.foo` never read from the entry is not composed. Its artifacts stay.

Skipping unchanged hashes is later.

### Writes name the unit

`write_output_file` / `remove_output_file` take `unit` (project-relative `.foo`). Dream checks the claim:

- the unit exists in the project;
- the unit is the entry or was read this run;
- the unit is not locked;
- the path is not stolen, project-owned, or unmanaged (unless `--fresh`).

Ownership is never inferred from the last read or from the entry.

When the session settles, Dream reconciles each unit that wrote this run.

### No provenance

Missing or empty `-o`: first writes register under the claimed unit.

`-o` has files and Dream has no provenance store: **error**. Pass `--fresh` or use an empty directory. Do not silently claim the tree.

### Target mismatch

The store is for one target. If it says `rust` and the user passes `-t go`: **error** unless `--fresh`.

### Where the store lives

A Dream-owned file in `-o` (not in the `.foo` tree): `.dream/provenance.json`. The Composer cannot write `.dream/`.

---

## `--fresh`

```bash
dream app.foo -t rust -o ./out --fresh
```

Reset this target realization and compose again. Ignore current provenance and target-specific locks.

Drop provenance, locks, and **Dream-owned** paths (unit + project), then delete `.dream/`. Leave unmanaged files (`README.md`, `logo.png`, …). Leftover toolchain dirs (`target/`, …) are not in the map, so they survive and make the dest occupied on the next no-store open — pass `--fresh` again or empty the folder. The user can `rm -rf` `-o` if they want an empty folder.

Not `--clean`. “Clean” already means “delete build artifacts” in too many toolchains.

---

## Locks (Pre-Gimbal)

Locks are **target-specific artifact locks**.

```bash
dream lock server.foo -t rust
```

Freezes the current accepted artifact set **and contents** for that unit in that target. Normal `dream` must not let the Composer mutate them.

Source-oriented: the user locks the `.foo` file, not each generated path.

To recompose a locked unit: unlock it, then run normal `dream`. Or `--fresh`.

Do not invent target-independent locked meaning before Gimbal.

### Reading a locked unit

Do **not** replace the `.foo` with the generated files. `read_source_file` always returns that unit’s foocode. Locking does not change what a source unit is.

A lock means: do not run the Composer on that unit. Writes to its artifacts are rejected.

Dependents still request the `.foo`. If they only get foocode, they will invent an API that does not match the frozen realization. If they only get `.rs`, they lose the unit.

**Compose mode:** if the provenance store already has a realization for this target, the read returns both — same graph edge:

```text
path:    users/active.foo
source:  <foocode>
locked:  true | false
artifacts:
  - src/users/active.rs
    <contents>
```

The model may read those artifacts. It may not write locked ones. No extra tool.

**`dream now`:** locks are `-t`-specific. The interpreter has no artifact set. It gets `{ path, source }` only. No compose.

After Gimbal, a lock should freeze formal meaning; provenance still tracks the translated files.

---

## Project Layer

Project-owned files (`Cargo.toml`, `go.mod`, `pyproject.toml`) are **not** freeform LLM rewrites.

### `set_dependencies`

One tool. It takes `unit` (project-relative `.foo`) plus that unit’s full dependency list. Dream checks the same ownership as a write: the unit exists and is the entry or was read this run. The list **replaces** that unit’s dependencies when the session settles. Units that do not call it keep their previous list.

Each entry is a **package name** plus optional **features**. Dream chooses the version. The model does not pass semver.

Project deps are the union of every unit’s list. Dream retracts a dep only if Dream installed it and no unit still lists it. User-added deps with no unit listing stay.

No `list_*` / `add_*` / `remove_*` dep tools. Do not call them imports. The composer does not read `Cargo.toml`.

Repair rejects `set_dependencies`.

### Package name

On first init of a known builder, Dream sets the package name from the **entry file stem** (`multifile.foo` → `multifile`). Not a tool. Later reconciles do not overwrite an existing name (a user rename stays).

### How files find each other in the target

Dream does not generate language-specific wiring (`mod`, `import`, `package`, crate roots, …). That is ordinary target source. The unit that owns the program entry (usually the Dream entry’s artifacts) writes whatever the target needs to see the other units’ files. Those paths come from reads and writes in the same session.

### Unknown / `unsupported` targets

No project layer and no `set_dependencies`. All Composer writes are unit-owned. If the model writes `go.mod` / `CMakeLists.txt`, the **first writer** owns that path. A second unit cannot steal it. Dream does not interpret those files.

---

## Repair

Build runs **after** the composition session settles. Repair does not pick a unit by guessing from rustc paths. The owner of an existing path is the map, not the model.

It is a new session with the toolchain diagnostics. Same builder. Writes stay in `-o`.

Allowed: overwrite a path that provenance already assigns to an **unlocked** unit. The owner is the map, not the model.

Rejected:

- new paths (repair does not grow the artifact set);
- locked artifacts;
- project-owned files;
- unmanaged files;
- `set_dependencies` (no current unit).

If the project needs a new file or a dep change, that is a normal `dream` (or `--fresh`), not repair. Repair only fixes what was already composed.

---

## Lock vs the filesystem

Record the unit’s **source hash** when locking.

| Situation | Normal `dream` |
|---|---|
| Locked `.foo` source hash changed | **Error.** Unlock to accept the new meaning, or restore the `.foo`. Do not recompose. Do not keep pretending the frozen files match. |
| Locked artifact **deleted** | **Error.** Restore the file, unlock, or `--fresh`. Do not regenerate it from foocode (that would re-dream a lock). |
| Locked artifact **edited** by hand | **Leave it.** The Composer still will not touch it. The lock blocks Dream, not the user. |

`--fresh` still ignores locks.

---

## Relationship to Gimbal

Now:

```text
.foo unit
    ↓
Composer
    ↓
0..N target artifacts
    ↓
target-specific lock
```

Later:

```text
.foo unit
    ↓
semantic elaboration
    ↓
Gimbal
    ↓
target translator
    ↓
0..N target artifacts
```

Provenance remains. Only what a lock *means* changes.

See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]]. Do not start that work here.

---

## Related

- [[Projects/Dream/Core Rules|Core Rules]]
- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Targets and Composition|Targets and Composition]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Semantic Locking and Inspection|Semantic Locking and Inspection]]
- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Implementation Plan|Implementation Plan]]
