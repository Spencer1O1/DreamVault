**Status:** Preliminary  
**Purpose:** Define Dream's two execution modes, CLI flags, target selection, and configuration.

---

## Project Invocation

Dream should eventually support both:

```bash
dream main.foo -t rust
```

and:

```bash
dream . -t rust
```

When passed a project directory, Dream resolves the configured entrypoint.

## Primary CLI

Dream has two main execution modes:

```bash
dream [OPTIONS] <FILE>
```

and:

```bash
dream now [OPTIONS] <FILE>
```

Default mode creates a target project.

`now` directly interprets the Dream program.

## Default Mode

Running:

```bash
dream server.foo -t rust -o ./out
```

means:

1. resolve the Dream source graph;
2. gather required semantic units;
3. determine their intended semantics;
4. compose a complete Rust project;
5. write it to `./out`;
6. build it when appropriate;
7. optionally run it.

The generated project is a first-class artifact.

## Immediate Mode

Running:

```bash
dream now main.foo
```

means:

1. resolve the Dream source graph;
2. gather all required `.foo` units;
3. execute the resolved program directly using the LLM interpreter;
4. return observable program output.

No target language is involved.

## Why `now`

`dream now` preserves the original Dream idea:

```text
pseudocode
    ↓
LLM
    ↓
program output
```

It is useful for:

- experiments;
- tiny programs;
- prototypes;
- learning;
- quick scripts;
- testing Dream semantics.

It is intentionally less deterministic than normal composition.

## Target Selection

Target selection uses:

```bash
-t
```

or:

```bash
--target
```

Examples:

```bash
dream app.foo -t rust
dream app.foo -t go
dream app.foo -t python
dream app.foo -t c
dream app.foo -t cpp
dream app.foo -t swift
dream app.foo -t kotlin
dream app.foo -t zig
```

Targets are open-ended strings.

These should also be valid:

```bash
dream app.foo -t "C++20"
dream app.foo -t "Common Lisp"
dream app.foo -t "POSIX shell"
```

Dream should not define valid target languages through a hardcoded enum.

## Generation vs Building

Dream distinguishes between:

1. target-project composition;
2. target-project building.

A target may be composable even when Dream does not know how to build it locally.

Example:

```bash
dream hello.foo -t cobol -o ./out
```

may successfully produce a COBOL project even if Dream cannot locate or safely invoke a COBOL toolchain.

## Output Directory

Compilation accepts:

```bash
-o
```

or:

```bash
--output
```

Example:

```bash
dream server.foo -t rust -o ./out
```

Possible output:

```text
out/
├── Cargo.toml
├── Cargo.lock
├── src/
│   └── ...
└── target/
```

The project should resemble ordinary hand-maintainable source code.

## Generated Projects Are First-Class

Dream should never treat generated projects as opaque temporary files.

A developer should be able to do:

```bash
dream app.foo -t rust -o ./app-rs

cd app-rs
nvim .
cargo build
```

and take over maintenance manually.

Dream can therefore bootstrap conventional software.

## `--run`

Example:

```bash
dream server.foo -t rust -o ./out --run
```

Pipeline:

```text
resolve
↓
compose
↓
build
↓
run
```

For interpreted Dream:

```bash
dream now server.foo
```

does not use `--run`; interpretation is already execution.

## `--no-build`

Future option:

```bash
dream app.foo -t rust -o ./out --no-build
```

Meaning:

> Compose the target project and stop.

Useful when:

- inspecting generated code;
- using unsupported toolchains;
- building manually;
- debugging composition.

## `--release`

Targets with release profiles may support:

```bash
dream app.foo -t rust --release
```

The Builder maps this to target-native behavior such as:

```bash
cargo build --release
```

## Configuration

Initial `.env`:

```env
OPENAI_API_KEY=...
DREAM_MODEL=...
```

Possible future:

```env
DREAM_DEFAULT_TARGET=rust
```

Do not prematurely build a generalized provider abstraction.

## Related

- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Targets and Composition|Targets and Composition]]
- [[Projects/Dream/Semantics and Strictness|Semantics and Strictness]]
