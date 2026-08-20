**Status:** Preliminary staged plan  
**Purpose:** Reach a working Dream language without building later semantic machinery too early.

The crate is a **separate workspace**, not this notes vault. The v0 contract is [[Projects/Dream/MVP|MVP]].

---

## Initial Rust Layout

Start small:

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
│   ├── project.rs
│   ├── unit.rs
│   ├── resolver.rs
│   └── files.rs
├── interpreter/
├── composer/
├── builder/
├── runner/
├── semantics/
└── cache/
```

The `semantics` and `cache` layers should not be built until they are required.

## MVP Phase 1 — Interpreter

Implement:

```bash
dream now main.foo
dream now --strict main.foo
```

Requirements:

- CLI parsing;
- `.env`;
- OpenAI tool loop;
- `list_source_files` and `read_source_file`;
- `stdout` and `stdin` tools;
- discard chat text;
- stricter prompt for `--strict`;
- useful `DreamError`s.

Single-file support is acceptable for the first spike if the source tools already exist.

## MVP Phase 2 — Multi-File Resolution

Use the source tools for real projects.

Implement:

- project root;
- path normalization;
- sandboxed list/read;
- request-loop cycle detection;
- recorded dependency sets.

Each `.foo` file is one semantic unit from this phase onward.

## MVP Phase 3 — Composition

Implement:

```bash
dream main.foo -t rust -o ./out
```

Pipeline:

```text
Dream units
    ↓
Composer (write_output_file)
    ↓
replace -o
```

No build. Open-ended `-t`.

## MVP Phase 4 — Build and Run

Implement:

```bash
dream main.foo -t rust -o ./out --build
dream main.foo -t rust -o ./out --run
```

`--run` implies build. Forward standard process IO.

## MVP Phase 5 — Known Builders

Add common toolchain support.

Generation remains open-ended.

## MVP Phase 6 — Semantic Cache Foundations

Add file-level semantic metadata infrastructure.

At minimum:

```text
canonical path
source hash
semantic status
```

Do not yet attempt sub-file granularity.

Do not wait on a formal semantic core for this phase.

## MVP Phase 7 — Bounded Repair

Add build diagnostics and Composer repair.

Keep the repair loop explicitly bounded.

## Later — Semantic Locking

Allow semantic meaning for a `.foo` file to be persisted.

See [[Projects/Dream/Semantic Locking and Inspection|Semantic Locking and Inspection]].

## Later — Incremental Re-Dreaming

When source changes:

```text
detect changed .foo units
↓
invalidate only those units
↓
re-dream them
↓
compare semantic interfaces
↓
propagate invalidation only when necessary
```

This should eventually make Dream practical for large projects.

See [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]].

## Later — Formal Semantic Representation

Once a formal semantic core exists, Dream may elaborate each `.foo` unit into a corresponding formal fragment.

That future architecture is not decided yet. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

A later deterministic `dream now` runtime is also not decided yet. See [[Projects/Dream/Later Interpreter Runtime|Later Interpreter Runtime]]. Do not start that work during the MVP phases.

## Related

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Core Rules|Core Rules]]
