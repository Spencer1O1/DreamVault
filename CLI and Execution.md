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

**v0 (implemented):** write a complete tree through `write_output_file`, then replace `./out`.

**Next:** declare the builder first, then reconcile into the existing `./out` project. Unlocked units may be recomposed. Locked units and unmanaged files stay. There is no `redream` command. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

It does not build unless `--build` or `--run` is passed.

Compose prints each tool call on stderr (name and path, not file contents). `dream now` does not.

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

may successfully produce a COBOL project. After compose settles, Dream asks for a builder; COBOL should be `unsupported`. `--build` / `--run` then do not run.

The builder is the declared toolchain, not a guess from `-t` or from the output tree. **Next:** declare it before any output writes. See [[Projects/Dream/Targets and Composition|Targets and Composition]].

If that toolchain is missing locally, `--build` / `--run` error and tell the user how to install it. Dream does not install it. The composed `-o` folder is already there.

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

**v0:** `-o` **replaces the whole folder**. Dream stages the new tree, then swaps it in, so a failed compose does not half-wipe the destination.

**Next:** `-o` is an existing project. Normal `dream` reconciles in place (compose stack from the entry). Unknown files are user-owned. Files in `-o` but no provenance → error. `-t` disagrees with the store → error unless `--fresh`.

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
compose / reconcile
↓
build
```

v0 still `replace -o` between compose and build.

Default `dream` does not build.

A failed build may go back to the composer a bounded number of times. A failed run does not. Repair must obey artifact ownership.

`--no-warn` treats toolchain warnings as a failed build, so they repair too. It is not `--strict`.

## `--run`

```bash
dream server.foo -t rust -o ./out --run
```

Pipeline:

```text
compose / reconcile
↓
build
↓
run
```

`--run` implies build.

`dream now` does not use `--run`. Interpretation is already execution.

## `--fresh`

Not in v0. Next contract:

```bash
dream app.foo -t rust -o ./out --fresh
```

Reset Dream’s realization: drop provenance, locks, and Dream-owned paths; leave unmanaged files; compose again. Ignores target-specific locks. Not `--clean` (that means “delete build artifacts” in too many toolchains).

## Locks

Not in v0. Next:

```bash
dream lock server.foo -t rust
```

Freezes that unit’s current target artifact set. Normal `dream` does not mutate those files. No `redream` command.

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
- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Targets and Composition|Targets and Composition]]
- [[Projects/Dream/Semantics and Strictness|Semantics and Strictness]]
