**Status:** Preliminary  
**Purpose:** Define target-aware composition, open-ended targets, builders, and repair.

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

## Known Builders

`-t` is an open-ended compose hint. A **builder** is a toolchain Dream will actually exec (`cargo`, `go`, …), not a language vibe (`cpp`, `embedded`).

Ask **before** any output writes. Known builders later unlock Dream’s project layer. No pick, or `unsupported`, means Dream will not `--build`, `--run`, or repair. Composition may still succeed (generic unit-owned writes).

Do not infer the builder from the output tree. Do not take build or run argv from the model. Do not put `set_builder` in the write-loop catalog.

This list does not constrain `-t`. Vague targets (Arduino, COBOL, …) stay `unsupported` until that builder exists.

If a known builder is declared but that toolchain is not installed, Dream returns an error with a short install hint. It does not install the toolchain. `unsupported` means Dream has no builder; a missing `cargo` means the user does not have it.

## Generated Build Metadata

Do not do this. A JSON `build` / `run` line from the model is shell access. Unknown targets are `unsupported`.

## Composition and Building Stay Separate

Core invariant:

> **The Composer realizes units. Dream owns project infrastructure. The Builder invokes toolchains.**

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

Repair must use an explicit maximum number of attempts. The builder is not asked again. After provenance exists: repair runs with the stack empty; it may only overwrite existing unlocked unit-owned paths (owner from the map). No new files, no `set_dependencies`. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

## Generated Project Is Not the Semantic IR

A Rust, Go, or Python output project is a target representation.

It is not Dream's formal semantic representation.

Pre-Gimbal architecture:

```text
Dream units
    ↓
builder first
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
