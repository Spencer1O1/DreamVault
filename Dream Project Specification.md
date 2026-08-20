## 1. Overview

**Dream** is a programming language for executable pseudocode.

A Dream program is written in `.foo` files using clear, informal programming notation rather than a rigid grammar.

Dream uses an LLM to determine what each source unit means, then either:

```bash
dream program.foo
```

compose that Dream program into a conventional software project in a requested target language, build it when possible, and optionally run it;

or:

```bash
dream now program.foo
```

execute the Dream program immediately using an LLM as the interpreter.

The core idea is:

> **Dream is executable pseudocode.**

Dream is not primarily an AI coding assistant.

The programmer writes the program.

Dream interprets the programmer's notation.

---

# 2. Fundamental Semantic Rule

The most important structural rule in Dream is:

> **One `.foo` file is one semantic unit.**

Dream does not attempt to deterministically discover function, declaration, or semantic boundaries inside an arbitrary pseudocode file.

The filesystem provides those boundaries.

A `.foo` file is the smallest independently interpreted, cached, locked, invalidated, and re-dreamed unit.

This rule is foundational to Dream's long-term architecture.

---

# 3. Why Files Are Semantic Units

Dream intentionally allows fuzzy syntax.

That means code such as:

```text
take the active users

sort them oldest first

return the first five
```

may not contain deterministic parser-visible function boundaries.

Dream therefore should not attempt to ask:

> Where does one semantic unit end and another begin?

Instead, the programmer expresses that structure through files.

```text
users/
├── active.foo
├── oldest.foo
└── top_five.foo
```

Each file can be interpreted independently.

This gives Dream deterministic:

- unit identity;
    
- dependency boundaries;
    
- cache keys;
    
- lock boundaries;
    
- invalidation boundaries;
    
- re-dream boundaries.
    

---

# 4. Semantic Unit Kinds

A semantic unit does not necessarily mean a function.

A `.foo` file may represent:

- a function;
    
- a value;
    
- a constant;
    
- a type;
    
- a data model;
    
- a transformation;
    
- a service;
    
- an executable entrypoint;
    
- another independently meaningful program component.
    

The important invariant is:

> **Each `.foo` file should describe one externally meaningful semantic thing.**

Example function:

```text
// users/active.foo

input is a list of users

return only the users whose active field is true
```

Example type:

```text
// models/user.foo

a User has:
    name: text
    age: integer
    active: boolean
```

Example entrypoint:

```text
// main.foo

import "./users/load.foo"
import "./users/active.foo"

users = load()
users = active(users)

print each user's name
```

---

# 5. Source Extension

Canonical Dream source files use:

```text
.foo
```

Examples:

```text
main.foo
load_users.foo
server.foo
user.foo
```

The extension is intentionally unrelated to the language name.

Keep it.

---

# 6. Project Structure

A Dream project is fundamentally a graph of semantic units.

Example:

```text
my-project/
├── main.foo
├── models/
│   └── user.foo
├── users/
│   ├── load.foo
│   ├── active.foo
│   └── oldest.foo
└── dream.toml
```

Each `.foo` file is independently addressable.

Imports define relationships between those units.

---

# 7. Deterministic vs Fuzzy Structure

Dream deliberately places deterministic structure around fuzzy semantics.

```text
DETERMINISTIC
────────────────────────
project root
file paths
.foo unit identity
imports
dependency graph
configuration

FUZZY
────────────────────────
meaning of each .foo file
```

After semantic interpretation, Dream should eventually return to a deterministic representation:

```text
.foo source
    ↓
semantic interpretation
    ↓
formal semantic representation
```

Long-term, that formal representation is intended to be Gimbal.

---

# 8. Dream Source Style

Dream should encourage pseudocode that still looks like programming.

Good:

```text
users = load users

for user in users:
    if user is active:
        print user.name
```

Also good:

```text
take all active users

print each one's name
```

Less ideal:

```text
Please make me an app that loads some users and displays them somehow.
```

Dream may understand prose-heavy source, but the language should encourage clear program intent rather than generic prompting.

---

# 9. Recommended Style

Documentation should prefer readable pseudocode such as:

```text
import "./users/active.foo"

users = load users

users = active(users)

for user in users:
    print user.name
```

Recommended syntax is a convention.

It does not imply a complete parser grammar.

---

# 10. Comments

Canonical comments use:

```text
// comment
```

Example:

```text
// only return accounts visible to the current user

return visible accounts
```

Other styles may sometimes be understood, but `//` is the official form.

---

# 11. Imports

Imports are one of the intentionally formal parts of Dream.

Canonical syntax:

```text
import "./users/active.foo"
```

Possible alias syntax:

```text
import "./users/active.foo" as active_users
```

Exact namespace rules may evolve, but import paths should remain structurally deterministic.

Imports operate on whole semantic units.

---

# 12. Import Semantics

If:

```text
main.foo
```

contains:

```text
import "./users/active.foo"
```

then `main.foo` depends on the semantic unit represented by:

```text
users/active.foo
```

This relationship is part of Dream's source dependency graph.

Dream should never need an LLM to determine whether the import exists or what file it refers to.

---

# 13. Source Dependency Graph

Example:

```text
main.foo
├── users/load.foo
├── users/active.foo
│   └── models/user.foo
└── users/oldest.foo
```

The graph is deterministic.

The semantics inside each node may initially require an LLM.

This distinction is central to Dream.

---

# 14. Project Root

Dream should identify a project root.

Eventually projects may use:

```text
dream.toml
```

Example:

```toml
[project]
name = "my-project"
entry = "main.foo"
```

Potential future project settings:

```toml
[project]
name = "my-project"
entry = "main.foo"
default_target = "rust"
```

A manifest is not required for the earliest prototype.

---

# 15. Project Invocation

Dream should eventually support both:

```bash
dream main.foo -t rust
```

and:

```bash
dream . -t rust
```

When passed a project directory, Dream resolves the configured entrypoint.

---

# 16. Primary CLI

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

---

# 17. Default Mode

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

---

# 18. Immediate Mode

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

---

# 19. Why `now`

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

---

# 20. Target Selection

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
dream app.foo -t brainfuck
```

Dream should not define valid target languages through a hardcoded enum.

---

# 21. Generation vs Building

Dream distinguishes between:

1. target-project composition;
    
2. target-project building.
    

A target may be composable even when Dream does not know how to build it locally.

Example:

```bash
dream hello.foo -t cobol -o ./out
```

may successfully produce a COBOL project even if Dream cannot locate or safely invoke a COBOL toolchain.

---

# 22. Output Directory

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

---

# 23. Generated Projects Are First-Class

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

---

# 24. The Source Resolver

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

---

# 25. Source Resolution Must Be Deterministic

Dream should not ask the model:

> What files do you think this program imports?

Canonical imports should be mechanically discoverable.

This ensures the project dependency graph can be constructed before semantic interpretation.

---

# 26. Source Access Tool

The semantic system may receive source through a constrained tool such as:

```text
read_source_file(path)
```

Dream validates the path and returns only allowed project source.

The model does not get arbitrary filesystem access.

---

# 27. No Semantic Execution During Discovery

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

---

# 28. The Composer

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
    

---

# 29. Why Composer

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

---

# 30. The Builder

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

---

# 31. The Runner

The **Runner** executes a built or runnable target artifact.

It should forward:

- stdin;
    
- stdout;
    
- stderr;
    
- exit status.
    

---

# 32. Core Execution Architecture

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

---

# 33. `--run`

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

---

# 34. `--no-build`

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
    

---

# 35. `--release`

Targets with release profiles may support:

```bash
dream app.foo -t rust --release
```

The Builder maps this to target-native behavior such as:

```bash
cargo build --release
```

---

# 36. Strict Mode

Both modes support:

```bash
--strict
```

Examples:

```bash
dream --strict app.foo -t rust
```

```bash
dream now --strict app.foo
```

Strict mode changes semantic inference.

It does not introduce a separate source grammar.

---

# 37. Default Ambiguity Handling

Normal Dream should resolve reasonable ambiguity.

Example:

```text
sort the users
```

If context strongly implies a conventional ordering, Dream may choose it.

Normal Dream should not demand specification of every obvious detail.

---

# 38. Strict Ambiguity Handling

Strict Dream should reject important ambiguity.

Example:

```text
sort the users
```

Possible result:

```text
DreamError: ambiguous ordering for User.
Specify a field or ordering criterion.
```

Strict mode should avoid:

- creative guesses;
    
- invented data;
    
- invented effects;
    
- unjustified assumptions.
    

---

# 39. Semantic Unit Identity

A semantic unit is identified primarily by its canonical project-relative path.

Example:

```text
users/active.foo
```

This identity is deterministic.

The path can act as the stable structural key for:

- semantic caching;
    
- locks;
    
- diagnostics;
    
- dependency metadata;
    
- invalidation.
    

---

# 40. Source Hashing

Each semantic unit should eventually have a source hash.

Conceptually:

```text
UnitKey:
    canonical_path
    source_hash
```

This allows Dream to detect whether the source for that semantic unit changed.

No LLM is needed to answer that question.

---

# 41. Semantic Cache

Long-term, each `.foo` unit may have cached semantic metadata.

Conceptually:

```text
users/active.foo

source_hash
semantic_hash
formal_representation
public_interface
dependencies
frontend_version
strictness
assumptions
```

The cache should be attached to the file-level semantic unit.

---

# 42. Re-Dreaming

**Re-dreaming** means recomputing the semantic meaning of a specific `.foo` unit.

Conceptually:

```bash
dream redream users/active.foo
```

This should invalidate that semantic unit and derive its meaning again.

Because `.foo` files are semantic atoms, re-dreaming has a deterministic boundary.

---

# 43. Incremental Semantics

The long-term architecture should support incremental semantic invalidation.

Example:

```text
users/
├── active.foo
└── oldest.foo
```

If only:

```text
oldest.foo
```

changes, Dream should ideally reuse the accepted meaning of:

```text
active.foo
```

and only re-dream the changed unit.

---

# 44. Interface-Aware Invalidation

Suppose:

```text
oldest.foo
```

previously represented:

```text
List<User> → List<User>
```

and after re-dreaming still represents:

```text
List<User> → List<User>
```

Then dependents may not need semantic reinterpretation.

Conceptually:

```text
source changed?
    ↓ yes
re-dream unit
    ↓
semantic interface changed?
    ├── no  → dependents remain valid
    └── yes → invalidate dependents
```

This is the desired long-term incremental model.

---

# 45. File Granularity Is Deliberate

Dream should not initially attempt sub-file semantic caching.

Do not introduce:

- line-level semantic IDs;
    
- model-detected functions inside files;
    
- AST-node caches;
    
- fuzzy internal semantic boundaries.
    

The `.foo` file is the semantic atom.

This can be revisited only if a deterministic source syntax eventually provides smaller stable boundaries.

---

# 46. One Semantic Thing Per File

Dream should encourage:

```text
users/
├── load.foo
├── active.foo
├── oldest.foo
└── save.foo
```

rather than:

```text
users.foo
```

containing twenty unrelated functions.

This is not merely a style preference.

It improves:

- semantic caching;
    
- re-dreaming;
    
- dependency tracking;
    
- locking;
    
- model context size;
    
- source review;
    
- composability.
    

---

# 47. Entry Files

An executable entrypoint such as:

```text
main.foo
```

is itself one semantic unit.

It may depend on many imported units.

Its job is typically composition:

```text
import "./users/load.foo"
import "./users/active.foo"

users = load()
users = active(users)

print each user's name
```

---

# 48. Runtime Effects

A pure LLM interpreter cannot truthfully perform arbitrary external operations.

Dream source such as:

```text
read users.json
```

should not cause the model to invent file contents.

Future `dream now` should expose controlled runtime capabilities.

---

# 49. Runtime Capability Model

Potential tools:

```text
read_file
write_file
read_stdin
get_env
http_request
```

Dream performs the actual effect.

The Interpreter decides when the program requires it.

This changes:

```text
LLM pretends to execute
```

into:

```text
LLM controls a bounded runtime
```

---

# 50. Imports Are Not Runtime File Access

These are separate:

```text
import "./users/load.foo"
```

and:

```text
read "./users.json"
```

The first is source resolution.

The second is program execution.

Do not conflate them.

---

# 51. Target-Aware Composition

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

---

# 52. Arbitrary Targets

Generation should remain open-ended.

Dream should not reject:

```bash
dream main.foo -t cobol
```

simply because COBOL does not have a built-in adapter.

If the Composer can generate it, generation may succeed.

---

# 53. Known Builders

Dream may provide known Builder support for common ecosystems.

Initial useful set:

- Rust;
    
- Go;
    
- C;
    
- C++;
    
- Python;
    
- JavaScript;
    
- TypeScript;
    
- Java;
    
- Kotlin.
    

This list does not constrain Composer targets.

---

# 54. Generated Build Metadata

Unknown targets may eventually provide structured build metadata.

Conceptually:

```json
{
  "target": "cobol",
  "build": ["cobc", "-x", "src/main.cob", "-o", "app"],
  "run": ["./app"]
}
```

Dream must validate such metadata before executing anything.

---

# 55. Composition and Building Stay Separate

Core invariant:

> **The Composer writes the target project. The Builder invokes toolchains.**

The Composer should not receive unrestricted shell access.

---

# 56. Build Repair

Future composition may use bounded repair.

```text
compose project
↓
build/check
↓
toolchain diagnostics
↓
Composer repair
↓
build/check
```

Repair must use an explicit maximum number of attempts.

---

# 57. Generated Project Is Not the Semantic IR

A Rust, Go, or Python output project is a target representation.

It is not Dream's formal semantic representation.

Initial architecture:

```text
Dream units
    ↓
Composer
    ↓
Target Project
```

Long-term architecture introduces Gimbal before target generation.

---

# 58. Configuration

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

---

# 59. Errors

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

---

# 60. Security Principle

> **Dream owns capabilities. The model owns interpretation and composition.**

Dream controls:

- source access;
    
- filesystem boundaries;
    
- target output directories;
    
- build execution;
    
- process execution;
    
- environment variables;
    
- secrets;
    
- runtime tools.
    

The model should never receive unrestricted local-machine authority.

---

# 61. Initial Rust Layout

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

---

# 62. Later Rust Layout

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

---

# 63. MVP Phase 1 — Interpreter

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

---

# 64. MVP Phase 2 — Multi-File Resolution

Add canonical imports.

Implement:

- project root;
    
- path normalization;
    
- recursive imports;
    
- cycle detection;
    
- source graph construction.
    

Each `.foo` file is one semantic unit from this phase onward.

---

# 65. MVP Phase 3 — Rust Composition

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

---

# 66. MVP Phase 4 — Run Support

Implement:

```bash
dream main.foo -t rust -o ./out --run
```

Forward standard process IO correctly.

---

# 67. MVP Phase 5 — Arbitrary Targets

Allow arbitrary strings for:

```bash
-t
--target
```

Do not require target registration.

---

# 68. MVP Phase 6 — Known Builders

Add common toolchain support.

Generation remains open-ended.

---

# 69. MVP Phase 7 — Semantic Cache Foundations

Before implementing Gimbal integration, add file-level semantic metadata infrastructure.

At minimum:

```text
canonical path
source hash
semantic status
```

Do not yet attempt sub-file granularity.

---

# 70. MVP Phase 8 — Bounded Repair

Add build diagnostics and Composer repair.

Keep the repair loop explicitly bounded.

---

# 71. Later — Semantic Locking

Allow semantic meaning for a `.foo` file to be persisted.

Potential commands:

```bash
dream lock users/active.foo
dream unlock users/active.foo
dream redream users/active.foo
```

Exact CLI may change.

The underlying unit remains the `.foo` file.

---

# 72. Later — Incremental Re-Dreaming

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

---

# 73. Later — Gimbal Semantic Representation

Once Gimbal is mature enough, Dream should use Gimbal as its formal semantic target.

Each `.foo` semantic unit should eventually elaborate into a corresponding Gimbal fragment or declaration.

That future architecture is defined separately in the Long-Term Architecture document.

---

# 74. Core Rules

## Semantic Unit Rule

> **One `.foo` file is one semantic unit.**

## Source Structure Rule

> **Dream never depends on an LLM to discover semantic boundaries inside a `.foo` file.**

## Multi-File Rule

> **Imports define the deterministic dependency graph between semantic units.**

## Composition Rule

> **The Composer creates target projects. The Builder builds them.**

## Target Rule

> **Target generation is open-ended. Builder support is optional.**

## Capability Rule

> **Dream owns capabilities. The model owns interpretation.**

## Strictness Rule

> **Normal Dream resolves reasonable ambiguity. Strict Dream exposes important ambiguity.**

## Artifact Rule

> **Generated target projects are first-class outputs and remain useful without Dream.**

---

# 75. Final Architecture

```text
                       Dream Project
                            │
                            ▼
                     Source Resolver
                            │
                            ▼
                .foo Semantic Unit Graph
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

---

# 76. Final Definition

> **Dream is an executable pseudocode language where each `.foo` file is an independently meaningful semantic unit, and projects can either run immediately through an LLM or be composed into ordinary software projects in virtually any target language.**