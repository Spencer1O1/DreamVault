**Status:** Preliminary  
**Purpose:** Define Dream's current execution architecture: resolver, interpreter, composer, builder, and runner.

---

## Core Execution Architecture

```text
                         Dream Project
                              │
                              ▼
                       Source Resolver
                              │
                              ▼
                    Source Dependency Graph
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
             `now`                      default
          Interpreter                   Composer
                │                           │
                ▼                           ▼
         Program Output                Target Project
                                            │
                                            ▼
                                         Builder
                                            │
                                            ▼
                                      Built Artifact
                                            │
                                         --run
                                            │
                                            ▼
                                          Runner
```

This is the current architecture. A later formal semantic core is not part of it. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

## The Source Resolver

The **Source Resolver** owns project structure.

Responsibilities:

- identify the project root;
- locate the entrypoint;
- parse canonical imports;
- normalize paths;
- enforce project boundaries;
- detect missing imports;
- detect cycles;
- construct the source dependency graph;
- provide source contents to the semantic system.

The Source Resolver does not determine program meaning.

## Source Resolution Must Be Deterministic

Dream should not ask the model:

> What files do you think this program imports?

Canonical imports should be mechanically discoverable.

This ensures the project dependency graph can be constructed before semantic interpretation.

## Source Access Tool

The semantic system may receive source through a constrained tool such as:

```text
read_source_file(path)
```

Dream validates the path and returns only allowed project source.

The model does not get arbitrary filesystem access.

## No Semantic Execution During Discovery

Dream should maintain this invariant:

> **Source discovery completes before interpretation or composition begins.**

Do not:

```text
execute half of main.foo
↓
discover another file
↓
load it
↓
reinterpret previous execution
```

Instead:

```text
resolve source graph
↓
gather required units
↓
interpret or compose
```

## The Composer

The **Composer** is the LLM-powered component that converts resolved Dream semantics into a conventional target-language project.

Conceptually:

```text
Resolved Dream Project
        +
Target
        ↓
Composer
        ↓
Target Project
```

The Composer may choose:

- files;
- modules;
- dependencies;
- frameworks;
- target-native idioms;
- package layout;
- configuration;
- entrypoints.

## Why Composer

The term **Composer** is intentionally used instead of “compiler agent.”

The LLM is not merely performing mechanical lowering.

It may decide how an abstract Dream intent should best appear in the target ecosystem.

Example:

```text
make a JSON API server
```

Target Rust may use:

```text
Axum + Tokio + Serde
```

Target Go may use:

```text
net/http
```

Target Python may use:

```text
FastAPI
```

The Composer creates a complete target-native project.

## The Builder

The **Builder** operates after composition.

Responsibilities:

- identify the relevant local toolchain;
- validate build instructions;
- invoke the build process;
- capture diagnostics;
- locate resulting artifacts.

Examples:

```text
Rust → cargo build
Go → go build
C → clang/gcc/cc
Python → no native build required
```

The Builder does not decide Dream semantics.

## The Runner

The **Runner** executes a built or runnable target artifact.

It should forward:

- stdin;
- stdout;
- stderr;
- exit status.

## Errors

Basic format:

```text
DreamError: ...
```

Examples:

```text
DreamError: imported source `./users/foo.foo` does not exist.
```

```text
DreamError: import escapes project root.
```

```text
DreamError: ambiguous semantic meaning in `users/oldest.foo`.
```

```text
DreamError: target project failed to build.
```

## Related

- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Targets and Composition|Targets and Composition]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
