**Status:** Phase list only. Do not track checkboxes here.  
**Purpose:** v0 is implemented. Next work is pre-Gimbal artifact ownership, not a formal semantic core.

The crate is a **separate workspace**, not this notes vault. Implemented contract: [[Projects/Dream/MVP|MVP]]. Next contract: [[Projects/Dream/Artifact Ownership|Artifact Ownership]]. Progress is `docs/plan.md` in the crate.

---

## Initial Rust Layout

Start small (historical):

```text
dream/
├── Cargo.toml
├── .env.example
├── README.md
└── src/
    ├── main.rs
    ├── cli.rs
    ├── config.rs
    ├── error.rs
    ├── openai.rs
    ├── source.rs
    ├── interpret.rs
    ├── compose.rs
    ├── build.rs
    └── run.rs
```

## Later Rust Layout

As needed:

```text
src/
├── source/
├── interpreter/
├── composer/
├── builder/
├── provenance/    # unit → target artifacts; not an IR
└── project/       # known-builder manifest layer
```

Do not add a `semantics/` / Gimbal layer until that work is actually started.

## MVP Phases 1–6 — Done

Interpreter, multi-file, compose (replace `-o`), known builders (asked **after** writes), build/run, bounded repair. Progress: crate `docs/plan.md`.

## Phase 7 — Builder First

Ask `set_builder` once **before** output writes. `-t` stays an open-ended hint. `unsupported` / no pick → compose only.

See [[Projects/Dream/Targets and Composition|Targets and Composition]].

## Phase 8 — Provenance and In-Place Reconcile

Stop treating `-o` as disposable. Compose stack from the entry: writes belong to the current unit; `read_source_file` of an unlocked unsettled unit recurses. Persist unit → artifact paths in `-o`. Enforce ownership on write/delete. Unknown files stay. No store + files, or `-t` mismatch → error. `--fresh` drops Dream-owned paths only and ignores locks.

No one-`.foo`-to-one-file rule. No lock CLI yet. No project tools yet. No IR.

See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

## Phase 9 — Project Layer

Known builders: Dream owns manifests. `set_dependencies` is names plus optional features; Dream picks versions. Package name from entry stem on init only. Dream does not generate target-language wiring. `unsupported`: first writer owns manifest-shaped files.

## Phase 10 — Target-Specific Locks

```bash
dream lock server.foo -t rust
```

Freeze that unit’s current target artifact set and source hash. Normal `dream` skips it. Source hash mismatch or missing locked artifact → error. Hand-edited locked files stay. No `redream` command.

See [[Projects/Dream/Semantic Locking and Inspection|Semantic Locking and Inspection]].

## Later — Unchanged-Unit Skip

Source hash so unlocked units that have not changed are not sent to the Composer again.

See [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]].

## Later — Formal Semantic Representation

Once a formal semantic core exists, Dream may elaborate each `.foo` unit into a corresponding formal fragment. Locks become target-independent. Provenance still tracks translated files.

That future architecture is not decided yet. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

A later deterministic `dream now` runtime is also not decided yet. See [[Projects/Dream/Later Interpreter Runtime|Later Interpreter Runtime]].

## Related

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Core Rules|Core Rules]]
