**Status:** Preliminary staged plan  
**Purpose:** Reach a working Dream language without building later semantic machinery too early.

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
│   └── imports.rs
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
- OpenAI request;
- file reading;
- stdout;
- strict prompt;
- useful errors.

Single-file support is acceptable for the first spike.

## MVP Phase 2 — Multi-File Resolution

Add canonical imports.

Implement:

- project root;
- path normalization;
- recursive imports;
- cycle detection;
- source graph construction.

Each `.foo` file is one semantic unit from this phase onward.

## MVP Phase 3 — Rust Composition

Implement:

```bash
dream main.foo -t rust -o ./out
```

Pipeline:

```text
Dream Source Graph
    ↓
Composer
    ↓
Cargo Project
    ↓
Builder
```

## MVP Phase 4 — Run Support

Implement:

```bash
dream main.foo -t rust -o ./out --run
```

Forward standard process IO correctly.

## MVP Phase 5 — Arbitrary Targets

Allow arbitrary strings for:

```bash
-t
--target
```

Do not require target registration.

## MVP Phase 6 — Known Builders

Add common toolchain support.

Generation remains open-ended.

## MVP Phase 7 — Semantic Cache Foundations

Add file-level semantic metadata infrastructure.

At minimum:

```text
canonical path
source hash
semantic status
```

Do not yet attempt sub-file granularity.

Do not wait on a formal semantic core for this phase.

## MVP Phase 8 — Bounded Repair

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

## Related

- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Core Rules|Core Rules]]
