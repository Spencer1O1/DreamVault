**Status:** Preliminary  
**Purpose:** Define Dream's current execution architecture: resolver, interpreter, composer, builder, and runner.

---

## Core Execution Architecture

```text
                         Dream Project
                              │
                              ▼
                       Source Resolver
                    list_source_files
                    read_source_file
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
             `now`                      default
          Interpreter                   Composer
           stdout/stdin            write_output_file
                │                           │
                ▼                           ▼
         Program Output              replace -o
                                            │
                                      --build / --run
                                            │
                                            ▼
                                      Builder / Runner
```

This is the current architecture. A later formal semantic core is not part of it. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

## The Source Resolver

The **Source Resolver** owns project structure.

Responsibilities:

- identify the project root;
- locate the entrypoint;
- normalize paths;
- enforce project boundaries;
- list project `.foo` paths through `list_source_files`;
- serve `.foo` source through `read_source_file`;
- reject paths outside the project;
- detect missing files and cycles during requests;
- remember the discovered dependency set.

The Source Resolver does not determine program meaning.

It does not parse an import syntax.

## Source Access Tools

Both modes may use:

```text
list_source_files
read_source_file(path)
```

`list_source_files` returns project-relative `.foo` paths only. No contents.

`read_source_file` returns one unit if it is inside the project.

The model does not get arbitrary filesystem access.

Which files are needed is part of meaning. The model may list the project, then read what it needs. Dream records each successful `read_source_file` as a dependency.

See [[Projects/Dream/Projects and Imports|Projects and Imports]] and [[Projects/Dream/MVP|MVP]].

## Request Loop, Not a Prelude

Dream should not require a complete source graph before interpretation begins.

This is allowed:

```text
interpret main.foo
↓
model requests users/active.foo
↓
Dream serves it
↓
continue
```

Do not emit a finished target project, or treat a unit as fully dreamed, until that unit's source requests have settled.

A later run can skip the loop when the unit's source hash still matches a recorded dependency set.

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

It writes only through `write_output_file`, into a staging directory for `-o`. It does not get interpreter runtime tools. See [[Projects/Dream/Targets and Composition|Targets and Composition]].

## The Builder

The **Builder** operates after composition, and only when `--build` or `--run` is passed.

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
DreamError: requested source `users/foo.foo` does not exist.
```

```text
DreamError: source request escapes project root.
```

```text
DreamError: ambiguous semantic meaning in `users/oldest.foo`.
```

```text
DreamError: target project failed to build.
```

## Related

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Targets and Composition|Targets and Composition]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
