**Status:** Next contract (after v0)  
**Purpose:** Pre-Gimbal target artifact ownership, in-place reconciliation, and locks. Implementers should follow this plus [[Projects/Dream/Core Rules|Core Rules]]. Do not invent a target-independent IR.

v0 (`[[Projects/Dream/MVP|MVP]]`) still **replaces** `-o` and asks for the builder **after** writes. That is the implemented CLI. This note is what comes next.

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

Toolchain outputs (`target/`, `node_modules/`, `Cargo.lock`, …) are build state, not semantic artifacts. Policy may vary by target. Do not treat them as Composer output.

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

**Declare the builder before any output writes.** v0 asked after the write loop; that is wrong for target-aware project tools.

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

While composing `server.foo` it may create, update, or delete only artifacts owned by `server.foo` (including newly registered paths for that unit).

Reject:

- overwrite of another unit’s artifact;
- overwrite of a project-owned file (use a project tool);
- overwrite of an unmanaged file (unless a later explicit user grant exists).

The Composer must not silently steal ownership.

It may propose new paths for the current unit. After that unit settles, Dream records the new set. Paths that were owned by this unit and are gone from the new set may be deleted.

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
compose the entry (stack)
    ↓
reconcile project-owned state
    ↓
build if asked
```

Normal `dream` must **not** `rm -rf` `-o`.

### Compose stack

Dream starts a compose job for the entry `.foo`. That job is the **current unit**. Every `write_output_file` belongs to it. The model does not name the owner.

`read_source_file` during compose:

| That unit | What Dream does |
|---|---|
| Locked for this target | No nested job. Return foocode + frozen artifacts. |
| Already settled this run | No nested job. Return foocode + this run’s artifacts. |
| Unlocked and not settled | **Recurse:** finish composing that unit first, then return foocode + its new artifacts. |
| Cycle in the stack | `DreamError` (same as today’s source cycle). |

A `.foo` never reached from the entry is not composed. Its artifacts stay.

Skipping unchanged hashes is later.

### No provenance

Missing or empty `-o`: first writes register under the current unit.

`-o` has files and Dream has no provenance store: **error**. Pass `--fresh` or use an empty directory. Do not silently claim the tree.

### Target mismatch

The store is for one target. If it says `rust` and the user passes `-t go`: **error** unless `--fresh`.

### Where the store lives

A Dream-owned file in `-o` (not in the `.foo` tree). Exact format is not precious. The Composer cannot write it.

---

## `--fresh`

```bash
dream app.foo -t rust -o ./out --fresh
```

Reset this target realization and compose again. Ignore current provenance and target-specific locks.

Drop provenance, locks, and **Dream-owned** paths (unit + project). Leave unmanaged files (`README.md`, `logo.png`, …). The user can `rm -rf` `-o` if they want an empty folder.

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

**Compose mode:** if Dream already has a realization for this target (locked, or settled this run, or just finished a nested compose), the read returns both — same graph edge:

```text
path:    users/active.foo
source:  <foocode>
locked:  true | false
artifacts:
  - src/users/active.rs
    <contents>
```

The model may read those artifacts. It may not write locked ones. No extra tool.

**`dream now`:** locks are `-t`-specific. The interpreter has no artifact set. It gets `{ path, source }` only. No compose recurse.

After Gimbal, a lock should freeze formal meaning; provenance still tracks the translated files.

---

## Project Layer

Project-owned files (`Cargo.toml`, `go.mod`, `pyproject.toml`) are **not** freeform LLM rewrites.

### `set_dependencies`

One tool, for the **current unit** only. The list **replaces** that unit’s claim set when the job settles.

Each claim is a **package name** plus optional **features**. Dream (or `cargo add` / equivalent) chooses the version. The model does not pass semver.

Project deps are the union of claims. Dream retracts a dep only if Dream installed it from a claim and no unit still claims it. User-added deps with no claim stay.

No `list_*` / `add_*` / `remove_*` dep tools. Do not call them imports. The composer does not read `Cargo.toml`.

### Package name

On first init of a known builder, Dream sets the package name from the **entry file stem** (`multifile.foo` → `multifile`). Not a tool. Later reconciles do not overwrite an existing name (a user rename stays).

### How files find each other in the target

Dream does not generate language-specific wiring (`mod`, `import`, `package`, crate roots, …). That is ordinary target source. The unit that owns the program entry (usually the Dream entry’s artifacts) writes whatever the target needs to see the other units’ files. The composer already has those paths from the compose stack.

### Unknown / `unsupported` targets

No project layer and no `set_dependencies`. All Composer writes are unit-owned. If the model writes `go.mod` / `CMakeLists.txt`, the **first writer** owns that path. A second unit cannot steal it. Dream does not interpret those files.

---

## Repair

Build runs **after** the compose stack is empty. Repair is not a job on that stack and does not pick a unit by guessing from rustc paths.

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
