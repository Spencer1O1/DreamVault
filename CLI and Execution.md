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

Default mode **composes** a target project and stops.

It does not build or run unless asked.

`now` directly interprets the Dream program.

## Default Mode

Running:

```bash
dream server.foo -t rust -o ./out
```

means:

1. start from the entry `.foo` file;
2. determine meaning, listing and reading other `.foo` units as needed;
3. write a complete target project through `write_output_file`;
4. replace `./out` with the staged tree.

It does not build. Add `--build` or `--run` for that.

The generated project is a first-class artifact.

## Immediate Mode

Running:

```bash
dream now main.foo
```

means:

1. start from the entry `.foo` file;
2. execute it with the LLM interpreter, listing and reading other `.foo` units as needed;
3. print whatever the model sends through `stdout`.

The model's chat text is discarded.

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

`-o` **replaces the whole folder**.

Dream stages the new tree, then swaps it in, so a failed compose does not half-wipe the destination.

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

## `--build`

```bash
dream server.foo -t rust -o ./out --build
```

Pipeline:

```text
compose
↓
replace -o
↓
build
```

Default `dream` does not build.

## `--run`

```bash
dream server.foo -t rust -o ./out --run
```

Pipeline:

```text
compose
↓
replace -o
↓
build
↓
run
```

`--run` implies build.

`dream now` does not use `--run`. Interpretation is already execution.

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

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Targets and Composition|Targets and Composition]]
- [[Projects/Dream/Semantics and Strictness|Semantics and Strictness]]
