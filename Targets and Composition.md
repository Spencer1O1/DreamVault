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

If `-t` **is** a catalog name, that is the toolchain. Do not ask. The composer gets the same `set_toolchain` result (`docs`, `project`, `entrypoint.path`). Fuzzy hints (`rust`, `cobol`, `monkey_c`) still ask `set_toolchain` before any output writes. That resolver turn sees only the requested target, not the entry `.foo` file. After bind or resolve to a catalog row, the store records that row (`cargo`), not the hint. Compose sees the toolchain fact, not `Requested target`. A later `-t rust` on that store reuses `cargo` (no resolver turn).

`unsupported` is Dream’s exec slot, not a language. After resolve to no row, compose still sees `Requested target` (`monkey_c`). The store records that `-t` string, not the word `unsupported`. `--build` / `--run` fail: Dream has no catalog row. No repair. No argv from the model. Literal `-t unsupported` is the empty language (no files required).

Known toolchains later unlock Dream’s project layer. No catalog row means Dream will not `--build`, `--run`, or repair. Composition may still succeed (generic unit-owned writes).

Do not infer the toolchain from the output tree. Do not take build or run argv from the model. Do not put `set_toolchain` in the write-loop catalog. The `set_toolchain` result is `docs`, `setup`, wipe-only `project`, read-only `configure` / `build` / `run`, and `entrypoint.path`. Dream execs those argv from the catalog.

This list does not constrain `-t`. Vague targets (Arduino, COBOL, Monkey C, …) have no catalog row until one exists. Compose still uses the requested target.

If a known toolchain is declared but that program is not installed, Dream returns an error with a short install hint. It does not install it. `unsupported` means Dream has no catalog row; a missing `cargo` means the user does not have it. Python’s official names are `python`, `python3`, and `py` — not a user alias table.

Each catalog row lists the dest paths that toolchain owns: the manifest Dream writes, plus lockfiles and build dirs it creates. `--fresh` drops that whole list. Dream does not infer `Cargo.lock` / `target/` from path shape.

## Generated Build Metadata

Do not do this. A JSON `build` / `run` line from the model is shell access. Unknown targets are `unsupported`.

## Composition and Building Stay Separate

Core invariant:

> **The Composer realizes units. Dream owns project infrastructure. Dream execs the declared toolchain.**

The Composer should not receive unrestricted shell or `-o` access.

Writes name the owning `.foo`, except this row’s setup files. Dream execs the catalog. Unknown files stay. `--fresh` is the wipe. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

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

Repair must use an explicit maximum number of attempts. The toolchain is not asked again. After provenance exists: repair runs with the stack empty; it may overwrite existing unlocked unit-owned paths and this row’s setup files. No new unit files. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

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
