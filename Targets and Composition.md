**Status:** Preliminary  
**Purpose:** Define target-aware composition, open-ended targets, toolchains, and repair.

---

## Target-Aware Composition

The Composer knows the requested target.

Example Dream unit:

```text
make a small HTTP server
```

Rust composition may choose:

```text
Axum
```

Go may choose:

```text
net/http
```

Python may choose:

```text
FastAPI
```

The externally intended behavior should remain consistent while implementation follows target conventions.

## Arbitrary Targets

Generation should remain open-ended.

Dream should not reject:

```bash
dream main.foo -t cobol
```

simply because COBOL does not have a built-in adapter.

If the Composer can generate it, generation may succeed.

## Known Toolchains

`-t` is an open-ended compose hint. A **toolchain** is a catalog row Dream will actually exec (`cargo`, `go`, `python`), not a language vibe (`cpp`, `embedded`). It is not “this language compiles.” Python is a row with no compile step.

Ask **before** any output writes. Known toolchains later unlock Dream’s project layer. No pick, or `unsupported`, means Dream will not `--build`, `--run`, or repair. Composition may still succeed (generic unit-owned writes).

Do not infer the toolchain from the output tree. Do not take build or run argv from the model. Do not put `set_toolchain` in the write-loop catalog. The `set_toolchain` result tells the model how Dream execs that row: `run.argv`, optional `build.argv`, `project` for every dest path that row owns, and `entry` when Dream owns the dest path.

This list does not constrain `-t`. Vague targets (Arduino, COBOL, …) stay `unsupported` until that catalog row exists.

If a known toolchain is declared but that program is not installed, Dream returns an error with a short install hint. It does not install it. `unsupported` means Dream has no catalog row; a missing `cargo` means the user does not have it. Python’s official names are `python`, `python3`, and `py` — not a user alias table.

Each catalog row lists the dest paths that toolchain owns: the manifest Dream writes, plus lockfiles and build dirs it creates. `--fresh` drops that whole list. Dream does not infer `Cargo.lock` / `target/` from path shape.

## Generated Build Metadata

Do not do this. A JSON `build` / `run` line from the model is shell access. Unknown targets are `unsupported`.

## Composition and Building Stay Separate

Core invariant:

> **The Composer realizes units. Dream owns project infrastructure. Dream execs the declared toolchain.**

The Composer should not receive unrestricted shell or `-o` access.

Writes only unit-owned artifacts in place. A write names the owning `.foo`. Manifests go through target-aware project tools (Phase 9). Unknown files stay. `--fresh` is the wipe. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

The Composer does not get `stdout`, `stdin`, or data-file tools. Those belong to `--lucid`.

Default compose does not build. See `--build` and `--run` in [[Projects/Dream/CLI and Execution|CLI and Execution]].

## Build Repair

`--build` / `--run` may repair a failed **build** (not a failed run, not a missing toolchain, not `unsupported`). `--no-warn` treats toolchain warnings as a failed build.

```text
reconcile
↓
build
↓
toolchain diagnostics
↓
Composer repair (same ownership rules, writes stay in `-o`)
↓
build
```

v0 still `replace -o` before the first build.

Repair must use an explicit maximum number of attempts. The toolchain is not asked again. After provenance exists: repair runs with the stack empty; it may only overwrite existing unlocked unit-owned paths (owner from the map). No new files, no `set_dependencies`. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

## Generated Project Is Not the Semantic IR

A Rust, Go, or Python output project is a target representation.

It is not Dream's formal semantic representation.

Pre-Gimbal architecture:

```text
Dream units
    ↓
toolchain first
    ↓
per-unit Composer (0..N artifacts each)
    ↓
project-layer reconcile
    ↓
Target Project
```

A later architecture may introduce a formal semantic representation before target generation. That is not decided yet. Do not fake it. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

## Related

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Core Rules|Core Rules]]
