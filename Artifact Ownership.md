**Status:** Current compose contract through Phase 10.  
**Purpose:** Pre-Gimbal target artifact ownership, in-place reconciliation, and locks. Implementers should follow this plus [[Projects/Dream/Core Rules|Core Rules]]. Do not invent a target-independent IR.

[[Projects/Dream/MVP|MVP]] is the historical v0 (replace `-o`, toolchain after writes). The crate no longer does that.

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
2. Project-owned  — this toolchain’s setup files (composer-writable) plus lockfiles and build dirs (wipe only)
3. Unmanaged      — no Dream provenance; user-owned by default
```

Each catalog toolchain lists **setup** (composer may write: `Cargo.toml`, `go.mod`, …) and **project** wipe-only paths (lockfiles, build dirs). Both are project-owned. They are not unit artifacts. Do not guess names at scan time — the catalog is the list. With no provenance store, any file outside `.dream/` means `-o` is occupied (`--fresh` or an empty directory).

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

## Toolchain First

`-t` stays an open-ended compose hint.

**Declare the toolchain before any output writes.** That turn is `set_toolchain` only — no `dream_error`, no write tools. v0 asked after the write loop; that is gone.

The `set_toolchain` result is for the composer, not a knob. Dream execs configure / build / run from the catalog row. The result is `docs`, `setup` (writable project files), `project` (wipe-only), read-only `configure` / `build` / `run`, and `entrypoint.path`. Showing argv is allowed. Taking argv from the model is forbidden. Every catalog row sets `entry`. `{stem}` is the entry `.foo` stem, interpolated at the boundary (`hey-you.py`, `hey-you.go`, `hey-you.c`). Rows whose toolchain requires a fixed file keep that path (`src/main.rs`, `App.java`, `app.ml`, …). `unsupported` returns only the name.

```text
entry .foo
    ↓
set_toolchain (once)
    ↓
known toolchain → Dream may init / load project-owned state
unsupported     → compose still allowed; no --build / --run / repair
    ↓
reconcile units
    ↓
reconcile project-owned state
    ↓
--build / --run if asked
```

Do not infer the toolchain from the tree. Do not take argv from the model.

Known toolchains get structured project tools. Unknown / `unsupported` targets use a generic write fallback. **Ownership still applies.** Do not give the Composer unrestricted filesystem or shell access to support a weird `-t`.

---

## Composer Write Boundary

The Composer does not get unrestricted write access to `-o`.

A write names the `.foo` unit it belongs to. It may create, update, or delete only artifacts owned by that unit (including newly registered paths for it).

Reject:

- overwrite of another unit’s artifact;
- overwrite of a wipe-only project path;
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
declare toolchain
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

`read_source_file` never starts a compose job. It returns the foocode immediately, plus that unit’s artifacts if the provenance store already has them (last accepted realization, locked or not). `--lucid` returns `{ path, source }` only.

A `.foo` never read from the entry is not composed. Its artifacts stay.

Do not skip an unlocked unit because its source hash matches. Lock if it must not change. The composer may leave owned files alone.

### Writes name the unit

`write_file` / `remove_file` take `unit` (project-relative `.foo`). Dream checks the claim:

- the unit exists in the project;
- the unit is the entry or was read this run;
- the unit is not locked;
- the path is not stolen, wipe-only, or unmanaged (unless `--fresh`). Setup paths on this row do not take a unit.

Ownership is never inferred from the last read or from the entry. The write-loop preamble states that the composer writes setup files and must not write wipe paths. Write and remove tool text does not repeat that.

When the session settles, Dream reconciles each unit that wrote this run.

`write_file` is a whole-file replace. Targeted / LSP edits are later. See [[Projects/Dream/Later Composer Tools|Later Composer Tools]].

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

Drop provenance, locks, and **Dream-owned** paths (unit + project), then delete `.dream/`. Also drop every catalog project path in `-o` (manifests, lockfiles, build dirs), even if the current store does not list them — so `--fresh -t go` does not keep `Cargo.toml`, `Cargo.lock`, or `target/`. Leave unmanaged files (`README.md`, `logo.png`, …) and source files not in the map. Those leftovers make the dest occupied on the next no-store open. The user can `rm -rf` `-o` if they want an empty folder.

Not `--clean`. “Clean” already means “delete build artifacts” in too many toolchains.

---

## Locks (Pre-Gimbal)

Locks are **target-specific artifact locks**.

```bash
dream lock server.foo -t rust -o ./out
dream lock Cargo.toml -t rust -o ./out
```

Lock a `.foo` file to freeze the source files it produced. Lock a setup file by name. Same `dream lock`. Those locks are independent. Any file the composer can write can be locked. Normal `dream` must not let the Composer mutate locked files.

Source-oriented: the user locks the `.foo` file, not each generated path. Setup has no `.foo`; name the file.

To recompose a locked unit: unlock it, then run normal `dream`. Or `--fresh`.

Do not invent target-independent locked meaning before Gimbal.

### Reading a locked unit

Do **not** replace the `.foo` with the generated files. `read_source_file` always returns that unit’s foocode. Locking does not change what a source unit is.

A lock means: do not run the Composer on that unit. Writes to its artifacts are rejected.

Dependents still request the `.foo`. If they only get foocode, they will invent an API that does not match the frozen realization. If they only get `.rs`, they lose the unit.

**Compose `list_source_files`:** each path includes `locked` for this dest. `--lucid` stays paths only.

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

**`--lucid`:** locks are `-t`-specific. The interpreter has no artifact set. It gets `{ path, source }` only. No compose.

After Gimbal, a lock should freeze formal meaning; provenance still tracks the translated files.

---

## Project Layer

Setup files (`Cargo.toml`, `go.mod`, `package.json`) are **this row’s allowlist**. The composer writes them. They are not owned by a `.foo`. Lock them by name (`dream lock Cargo.toml`). Wipe-only paths (`target/`, `Cargo.lock`) are project-owned and not writable.

`read_file` may read unit-owned dest files and this row’s setup files.

### Package name

The composer chooses the name in the setup file. Dream interpolates `{stem}` only for `entrypoint.path`.

Python `--run` is `python {stem}.py` in `-o` (`my.foo` → `my.py`). Same stem. Dream does not rename a composed file. If that script is missing, run fails. Exec tries `python`, then `python3`, then `py`. The `set_toolchain` reply still says `python`.

### How files find each other in the target

Dream does not generate language-specific wiring (`mod`, `import`, `package`, crate roots, …). That is ordinary target source. The unit that owns the program entry (usually the Dream entry’s artifacts) writes whatever the target needs to see the other units’ files. Those paths come from reads and writes in the same session.

### Unknown / `unsupported` targets

Empty setup. All Composer writes are unit-owned. If the model writes `go.mod` / `CMakeLists.txt`, the **first writer** owns that path. A second unit cannot steal it. Dream does not interpret those files.

---

## Repair

Build runs **after** the composition session settles. A failed **pre-run** catalog step (configure or build) may repair. A failed run does not. A missing program does not.

It is a new session with an empty stack: diagnostics plus the toolchain fact. Same toolchain. Writes stay in `-o`. The store target is the catalog row (`cargo`), not a fuzzy `-t` hint (`rust`).

Allowed: overwrite a path that provenance already assigns to an **unlocked** unit; create or overwrite this row’s setup files when setup is not locked. The owner of an existing unit path is the map, not the model.

Rejected:

- new unit-owned paths;
- locked dest files;
- wipe-only project paths;
- unmanaged files.

If the project needs a new unit file, that is a normal `dream` (or `--fresh`), not repair.

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
