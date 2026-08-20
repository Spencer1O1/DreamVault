## 1. Overview

**Dream** is a programming language for executable pseudocode.

A Dream source file does not need to obey a rigid programming-language grammar. Instead, the programmer writes code in clear pseudocode, and Dream uses an LLM to determine what the program means.

Dream has two primary modes:

```bash
dream program.foo
```

This turns the Dream program into a conventional software project in a chosen target language, builds it when possible, and optionally runs it.

```bash
dream now program.foo
```

This executes the Dream program directly through an LLM without first generating a conventional source project.

The core idea is:

> **Dream lets syntax remain informal while making program behavior concrete as early as possible.**

Dream is not primarily an AI coding assistant.

It is a programming language with a semantic, LLM-powered frontend.

---

# 2. Short Definition

> **Dream is executable pseudocode.**

---

# 3. One-Sentence Definition

> **Dream is an executable pseudocode language that can either run immediately through an LLM or be composed into a conventional software project in virtually any target language.**

---

# 4. Core Philosophy

Traditional programming languages usually work like this:

```text
precise source syntax
        ↓
      parser
        ↓
       AST
        ↓
    semantics
        ↓
    executable
```

Dream moves the precision boundary later:

```text
informal pseudocode
        ↓
 semantic understanding
        ↓
 concrete program
```

For normal Dream execution:

```text
Dream source
     ↓
Source Resolver
     ↓
Composer
     ↓
Target Project
     ↓
Builder
     ↓
Built Artifact
```

For immediate execution:

```text
Dream source
     ↓
Source Resolver
     ↓
Interpreter
     ↓
Program Output
```

The core design principle is:

> **The programmer specifies intent. Dream determines how that intent becomes executable.**

---

# 5. Source Files

Canonical Dream source files use:

```text
.foo
```

Examples:

```text
main.foo
server.foo
math.foo
users.foo
```

The extension is intentionally unrelated to the language name.

Keep it.

---

# 6. Example Dream Program

```text
// server.foo

make a web server on port 8080

GET /hello should return:
    hello world

log every request
```

Generate and build it as Rust:

```bash
dream server.foo --target rust -o ./server
```

Possible output project:

```text
server/
├── Cargo.toml
├── Cargo.lock
└── src/
    └── main.rs
```

Generate the same Dream program as Go:

```bash
dream server.foo --target go -o ./server-go
```

Or Python:

```bash
dream server.foo --target python -o ./server-python
```

Or something less conventional:

```bash
dream server.foo --target "Common Lisp" -o ./server-lisp
```

Dream should not require a target language to be hardcoded before it can be used.

---

# 7. Dream Is Pseudocode, Not Prompting

Dream programs should still look like programs.

Good Dream:

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
Please make an application that gets some users and maybe displays their names.
```

Dream may still understand that, but the language should encourage **programmatic pseudocode**, not generic app-description prompting.

The framing should remain:

> **The programmer wrote the program.**

The LLM is interpreting the programmer's notation.

---

# 8. Surface Syntax

Dream intentionally permits multiple surface styles.

These may all mean the same thing:

```text
for every user:
    print their name
```

```text
print the name of each user
```

```text
users.each:
    show name
```

```text
go through the users and print names
```

Dream is many-to-one:

```text
many possible source phrasings
            ↓
      same program intent
```

The language should still document a recommended style for readability and consistency.

---

# 9. Recommended Style

Documentation should prefer clear pseudocode such as:

```text
import "./users.foo"

users = load users

for user in users:
    if user.age >= 18:
        print user.name
```

Recommended syntax is a convention, not necessarily a parser requirement.

---

# 10. Comments

Canonical comment syntax:

```text
// comment
```

Example:

```text
// don't include inactive users

users = only active users
```

Other comment styles may be understandable to the model, but `//` should be the official style.

---

# 11. Primary CLI

Dream has two primary forms:

```bash
dream [OPTIONS] <FILE>
```

and:

```bash
dream now [OPTIONS] <FILE>
```

Default mode creates a target project.

`now` executes immediately through the interpreter.

---

# 12. Default Mode

Running:

```bash
dream server.foo
```

means:

> Resolve the Dream project, understand its semantics, create a conventional software project in the selected target, build that project when appropriate, and leave the generated project on disk.

The generated project is a real output artifact.

It is not temporary compiler scratch space.

---

# 13. Immediate Mode

Running:

```bash
dream now server.foo
```

means:

> Resolve the Dream project and execute its semantics directly through the LLM without creating a target-language project first.

Example:

```text
name = "Spencer"

if name == "Spencer":
    print "based"
else:
    print "hello " + name
```

```bash
dream now hello.foo
```

Output:

```text
based
```

The LLM acts as the interpreter.

---

# 14. Why `now` Exists

`dream now` is the original absurd version of Dream:

```text
pseudocode
    ↓
LLM
    ↓
stdout
```

It is useful for:

- experimentation;
    
- tiny scripts;
    
- prototypes;
    
- learning;
    
- testing ideas;
    
- running programs without generating another project.
    

It is less deterministic than normal Dream execution.

---

# 15. Target Selection

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
dream server.foo -t rust
dream server.foo -t go
dream server.foo -t python
dream server.foo -t c
dream server.foo -t cpp
dream server.foo -t kotlin
dream server.foo -t swift
dream server.foo -t zig
dream server.foo -t lua
dream server.foo -t ocaml
```

Long form:

```bash
dream server.foo --target rust
```

The target argument should be an open-ended string.

These should also be valid:

```bash
dream app.foo -t "C++20"
dream app.foo -t "Java 25"
dream app.foo -t "POSIX shell"
dream app.foo -t "Common Lisp"
dream app.foo -t brainfuck
```

Dream should not define the set of valid target languages through an enum.

---

# 16. Target Philosophy

Dream distinguishes between:

1. **Target generation**
    
2. **Target building**
    

Target generation should be effectively open-ended.

If the Composer understands the requested target well enough to create a valid project, Dream may generate it.

Target building is separate.

Dream may have built-in knowledge of common ecosystems, but lack of a built-in builder must not mean a target is unsupported.

---

# 17. Example Build Knowledge

Dream may know conventional toolchains such as:

```text
Rust       → cargo
Go         → go
C          → clang / gcc / cc
C++        → clang++ / g++
Python     → python
JavaScript → node
TypeScript → tsc / tsx / project tooling
Java       → javac / Gradle
Kotlin     → kotlinc / Gradle
```

This knowledge exists for convenience.

It must not define which targets the Composer is allowed to generate.

---

# 18. Output Directory

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

Dream creates a normal project under:

```text
./out
```

Rust example:

```text
out/
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── main.rs
│   └── ...
└── target/
```

Python example:

```text
out/
├── pyproject.toml
├── src/
│   └── ...
└── ...
```

Go example:

```text
out/
├── go.mod
├── main.go
└── ...
```

---

# 19. Generated Projects Are First-Class Artifacts

A user should be able to do this:

```bash
dream app.foo -t rust -o ./app-rs

cd app-rs
nvim .
cargo build
```

and never use Dream again if they do not want to.

This is an important property.

Dream can bootstrap ordinary software.

Workflow:

```text
Dream pseudocode
      ↓
target project
      ↓
developer takes ownership
```

Dream should not hide generated source code.

---

# 20. The Composer

The **Composer** is the LLM-powered component that turns a resolved Dream project into a complete project for a requested target.

Conceptually:

```text
Resolved Dream Project
        +
Target
        +
Composition Policy
        ↓
      Composer
        ↓
Complete Target Project
```

The Composer may decide:

- project structure;
    
- source file organization;
    
- target-native idioms;
    
- libraries;
    
- dependencies;
    
- framework choices;
    
- configuration files;
    
- package manifests;
    
- module boundaries;
    
- entrypoints.
    

It is not called the compiler agent because its job is broader than conventional compilation.

It **composes a software project**.

---

# 21. Why “Composer”

The Composer is not simply translating tokens into another language.

For source like:

```text
make a JSON API server
```

and target:

```text
rust
```

it might choose:

- Cargo project layout;
    
- Axum;
    
- Tokio;
    
- Serde;
    
- route organization.
    

For:

```text
python
```

it might choose:

- `pyproject.toml`;
    
- FastAPI;
    
- application module layout.
    

For:

```text
go
```

it may prefer the standard library.

That work is closer to project composition than traditional code generation.

---

# 22. The Builder

The **Builder** is separate from the Composer.

Its job is to take the completed target project and perform the appropriate local build process.

Pipeline:

```text
Composer
    ↓
Target Project
    ↓
Builder
    ↓
Built Artifact
```

For Rust:

```text
Cargo project
    ↓
Builder
    ↓
cargo build
```

For Go:

```text
Go module
    ↓
Builder
    ↓
go build
```

For Python there may be no native compilation step.

---

# 23. The Runner

The **Runner** executes the generated program after a successful build or target-specific preparation.

```text
Built/Runnable Artifact
          ↓
        Runner
          ↓
    Program Process
```

The Runner should forward:

- stdin;
    
- stdout;
    
- stderr;
    
- exit code.
    

---

# 24. The Interpreter

The **Interpreter** powers:

```bash
dream now main.foo
```

It receives the fully resolved Dream source context and executes the program directly through the LLM.

Pipeline:

```text
Dream Project
    ↓
Source Resolver
    ↓
Resolved Source Graph
    ↓
Interpreter
    ↓
Program Output
```

The Interpreter is not involved in normal target-project generation.

---

# 25. The Source Resolver

The **Source Resolver** gathers the Dream files required by a project.

Responsibilities:

- identify the entrypoint;
    
- detect imports;
    
- resolve source paths;
    
- enforce project boundaries;
    
- detect import cycles;
    
- gather required `.foo` files;
    
- expose constrained source-file access to the LLM when appropriate;
    
- build the source dependency graph.
    

The same Source Resolver should be used by both the Composer and Interpreter.

---

# 26. Core Component Model

```text
                         Dream Project
                              │
                              ▼
                       Source Resolver
                              │
                              ▼
                    Resolved Source Graph
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

# 27. Multi-File Projects

Dream must support multi-file programs.

Example:

```text
my-project/
├── main.foo
├── math.foo
├── users.foo
└── util/
    └── formatting.foo
```

A Dream file may import another Dream file.

Example:

```text
import "./math.foo"
import "./users.foo"

numbers = users.load()
print math.average(numbers)
```

Imports are one area where Dream should intentionally have a small amount of formal syntax.

---

# 28. Canonical Import Syntax

Canonical form:

```text
import "./math.foo"
```

Nested path:

```text
import "./util/formatting.foo"
```

Dream may understand other forms eventually, but documentation should use one official syntax.

Project structure should be less ambiguous than ordinary Dream statements.

---

# 29. Dependency Graph

Dream should resolve only relevant source dependencies.

Example:

```text
main.foo
   │
   ├── math.foo
   │      └── numbers.foo
   │
   └── users.foo
```

Dream should not automatically dump every `.foo` file in the directory into model context.

The source graph should begin from the entrypoint.

---

# 30. LLM Source Access

The Composer and Interpreter should be able to request imported Dream source through a constrained Dream-owned tool.

Conceptually:

```text
read_source_file(path)
```

The LLM does not get arbitrary filesystem access.

Dream validates:

- the requested path;
    
- project boundaries;
    
- allowed file type;
    
- path normalization;
    
- file existence.
    

Then Dream returns the source contents.

---

# 31. Source Gathering

Example:

```text
main.foo
```

contains:

```text
import "./math.foo"
```

The agent requests:

```text
./math.foo
```

That source contains:

```text
import "./numbers.foo"
```

The agent requests:

```text
./numbers.foo
```

This continues until all required source context is gathered.

---

# 32. Source Resolution Invariant

> **No program execution or target-project composition begins until all required Dream source context has been gathered.**

Do not do this:

```text
start executing main.foo
↓
discover import later
↓
load new source
↓
reinterpret prior state
```

Instead:

```text
DISCOVERY
    ↓
complete source context
    ↓
INTERPRET or COMPOSE
```

The LLM may participate during discovery, but Dream should maintain a clean phase boundary.

---

# 33. Project Root

Dream should identify a project root.

Eventually a project may contain:

```text
dream.toml
```

Example:

```text
my-project/
├── dream.toml
├── main.foo
├── math.foo
└── users/
    └── model.foo
```

Possible configuration:

```toml
[project]
name = "my-project"
entry = "main.foo"
```

A manifest is not required for the first prototype.

---

# 34. Project Invocation

Eventually both should work:

```bash
dream main.foo -t rust
```

and:

```bash
dream . -t rust
```

When given a directory, Dream can use `dream.toml` or another entrypoint convention.

---

# 35. Import Security

Imports must not casually escape the Dream project root.

This should fail:

```text
import "../../../../etc/passwd"
```

Source imports are part of Dream's program structure.

They are not arbitrary filesystem access.

---

# 36. Source Imports vs Runtime Files

These are fundamentally different:

```text
import "./math.foo"
```

and:

```text
read "./users.json"
```

The first means:

> Load another Dream source module.

The second means:

> Perform a runtime file effect.

Source access belongs to the Source Resolver.

Runtime data access belongs to the program runtime.

Do not combine these capability models.

---

# 37. `--run`

Normal mode accepts:

```bash
--run
```

Example:

```bash
dream server.foo -t rust -o ./out --run
```

Meaning:

```text
resolve source
    ↓
compose project
    ↓
build
    ↓
run
```

For Python:

```bash
dream script.foo -t python -o ./out --run
```

may mean:

```text
compose Python project
    ↓
prepare dependencies if needed
    ↓
run Python entrypoint
```

---

# 38. `--no-build`

A useful option:

```bash
--no-build
```

Example:

```bash
dream server.foo -t rust -o ./out --no-build
```

Meaning:

> Compose the target project and stop there.

Useful for:

- code inspection;
    
- unsupported toolchains;
    
- debugging;
    
- manual builds;
    
- environments without local compilers.
    

---

# 39. `--release`

Targets with release profiles may support:

```bash
--release
```

Example:

```bash
dream app.foo -t rust -o ./out --release
```

The Builder maps this to target-native behavior such as:

```bash
cargo build --release
```

This option should be interpreted appropriately for each target.

---

# 40. Strict Mode

Both paths support:

```bash
--strict
```

Examples:

```bash
dream --strict app.foo -t rust
```

and:

```bash
dream now --strict app.foo
```

Strict mode changes semantic inference.

It does not introduce a separate syntax.

---

# 41. Default Ambiguity Handling

Normal Dream should resolve reasonable ambiguity.

Example:

```text
sort the users
```

If context strongly suggests an obvious ordering, Dream may choose it.

The goal is to avoid turning Dream into:

```text
DreamError: please specify every obvious thing.
```

Normal mode should be useful precisely because it can infer conventional intent.

---

# 42. Strict Ambiguity Handling

Strict mode should reject semantically important unresolved ambiguity.

Example:

```text
sort the users
```

Possible result:

```text
DreamError: ambiguous ordering for User.
Specify a field or ordering criterion.
```

Strict mode should:

- prefer literal interpretations;
    
- prefer conventional programming semantics;
    
- avoid creative guesses;
    
- avoid inventing unavailable data;
    
- avoid inventing side effects;
    
- reject important assumptions;
    
- produce clear diagnostics.
    

---

# 43. Strict Composition

For:

```bash
dream --strict app.foo -t rust
```

the Composer should only produce a target project if it can establish sufficiently unambiguous program intent.

Strict mode is the closest early Dream gets to conventional compile-time semantic certainty.

---

# 44. Target-Aware Composition

The Composer knows the requested target before creating the project.

Input conceptually includes:

```text
Target: rust
```

or:

```text
Target: python
```

Target choice may legitimately affect implementation decisions.

Dream:

```text
make a small HTTP server
```

Rust may become:

```text
Axum + Tokio
```

Go may become:

```text
net/http
```

Python may become:

```text
FastAPI
```

The semantics should stay equivalent while implementation follows the target ecosystem.

---

# 45. Dependency Selection

The Composer may choose appropriate target-language dependencies.

Example:

```text
make a JSON API server
```

For Rust, it might select:

```text
axum
serde
tokio
```

For Go, it might rely primarily on the standard library.

For Python, it might select FastAPI.

Dream should eventually support dependency policies, but the initial Composer may make reasonable choices automatically.

---

# 46. Future Composition Policies

Potential future options:

```text
prefer standard library
minimal dependencies
offline only
no external dependencies
prefer mature libraries
```

These are not required for v0.1.

---

# 47. Target Project Structure

The Composer produces a complete target-native project.

Rust:

```text
out/
├── Cargo.toml
├── src/
│   ├── main.rs
│   └── ...
└── ...
```

Java:

```text
out/
├── build.gradle.kts
├── settings.gradle.kts
└── src/
    └── main/
        └── java/
            └── ...
```

Python:

```text
out/
├── pyproject.toml
├── src/
│   └── ...
└── ...
```

The Composer should use idiomatic project structure for the requested target.

---

# 48. Source Structure Does Not Need Exact Target Mapping

Dream:

```text
main.foo
users.foo
math.foo
```

might become Rust:

```text
src/main.rs
src/users.rs
src/math.rs
```

But Dream should not require a one-to-one mapping.

The Composer may reorganize code if the target language calls for a different structure.

Dream source organization communicates architecture and intent, not exact target file placement.

---

# 49. Generation and Building Are Separate

Important invariant:

> **The Composer creates files. The Builder invokes toolchains.**

Do not allow the Composer to freely execute shell commands while composing the project.

Initial lifecycle:

```text
1. Resolve Dream source
2. Determine target
3. Compose complete target project
4. End composition
5. Validate generated project
6. Build
7. Optionally run
```

This separation keeps capabilities understandable.

---

# 50. Build Discovery

The Builder needs to know how to build a target project.

Two strategies can coexist.

## Known Builders

Dream contains explicit support for common ecosystems.

Examples:

```text
rust → cargo
go → go build
python → python
```

## Generated Build Metadata

For unknown targets, the Composer may provide structured build metadata.

Conceptual example:

```json
{
  "target": "cobol",
  "build": [
    "cobc",
    "-x",
    "src/main.cob",
    "-o",
    "app"
  ],
  "run": [
    "./app"
  ]
}
```

Dream must validate such metadata before invoking anything.

---

# 51. Unknown Targets

This should be valid:

```bash
dream main.foo -t cobol -o ./out
```

If Dream can compose the project but cannot safely determine how to build it:

```text
Generated COBOL project at ./out.

No confirmed local build procedure is available for target `cobol`.
The project was not built.
```

This is still a successful target generation.

Dream should not reject it as an unsupported language.

---

# 52. Brainfuck Is Technically Valid

This should also be valid:

```bash
dream hello.foo -t brainfuck -o ./out
```

Possible project:

```text
out/
└── main.bf
```

If no builder exists:

```text
Generated Brainfuck project at ./out.
No known local build/run procedure is configured for target `brainfuck`.
```

Dream's target model should remain open-ended.

---

# 53. Build Validation

Before invoking a toolchain, Dream should validate:

- expected output directory;
    
- generated file paths;
    
- target metadata;
    
- build executable;
    
- arguments;
    
- working directory;
    
- environment passed to the process.
    

Model-provided commands are untrusted.

---

# 54. Build Repair

A later feature may use compiler or toolchain diagnostics to repair generated projects.

Example:

```text
compose project
      ↓
cargo check
      ↓
Rust diagnostics
      ↓
Composer receives diagnostics
      ↓
patch generated project
      ↓
cargo check
```

This can dramatically improve successful builds.

It should be bounded.

---

# 55. Repair Loop Rules

Repair must not become an uncontrolled autonomous loop.

Example:

```text
maximum repair attempts: 3
```

The project on disk remains the source of truth.

Each repair attempt modifies that project.

Pipeline:

```text
Target Project
      ↓
Builder diagnostics
      ↓
bounded Composer repair
      ↓
updated Target Project
```

---

# 56. Generated Project Is Not the Dream IR

A generated Rust or Python project is not technically Dream's intermediate representation.

It is a target-specific lowered representation.

Current pipeline:

```text
Dream source
    ↓
Composer
    ↓
Target Project
```

A future architecture may introduce:

```text
Dream source
    ↓
Dream IR
    ↓
Composer / Target Generator
    ↓
Target Project
```

But this should not be built prematurely.

---

# 57. No Custom IR Initially

The first useful implementation does not need:

- a complete AST;
    
- formal static semantics;
    
- a Dream VM;
    
- deterministic code generators;
    
- a large type system;
    
- a custom optimizer.
    

The initial goal is to prove:

> Dream pseudocode can reliably become useful conventional software projects.

Direct composition is enough for that.

---

# 58. Future Dream IR

A semantic intermediate representation may become valuable later.

Possible architecture:

```text
Dream Source
    ↓
Semantic Frontend
    ↓
Dream IR
    ↓
Target Generator
    ├── Rust
    ├── Go
    ├── Python
    └── ...
```

Benefits:

- semantic locking;
    
- multiple targets from one interpretation;
    
- deterministic target generation;
    
- stronger validation;
    
- static analysis;
    
- type checking;
    
- lower API cost;
    
- reproducibility;
    
- model independence after interpretation.
    

Only add this when direct composition becomes limiting.

---

# 59. Future Semantic Locking

Potential command:

```bash
dream lock main.foo
```

This would record exactly what Dream believes the program means.

Then:

```bash
dream main.foo -t rust
dream main.foo -t go
dream main.foo -t python
```

could all derive from the same accepted semantics.

Conceptually:

```text
main.foo
   ↓
semantic interpretation
   ↓
Dream IR / semantic lock
   ├── Rust
   ├── Go
   └── Python
```

This is a long-term feature.

---

# 60. Why Semantic Locking Matters

LLM interpretation may vary due to:

- model updates;
    
- prompt updates;
    
- provider changes;
    
- nondeterminism;
    
- ambiguous source.
    

A semantic lock could provide:

- reviewable inferred meaning;
    
- stable recompilation;
    
- cross-target consistency;
    
- reduced model calls;
    
- protection from interpretation drift.
    

---

# 61. Future `inspect`

Possible command:

```bash
dream inspect main.foo
```

Example output:

```text
Imports:
  ./users.foo
  ./math.foo

Interpretations:
  "active users" → user.active == true
  "oldest first" → descending by user.age

Ambiguities:
  none
```

Another possibility:

```bash
dream inspect main.foo --imports
```

Output:

```text
main.foo
├── users.foo
│   └── database.foo
└── math.foo
```

---

# 62. Immediate Interpreter Behavior

For:

```bash
dream now main.foo
```

Dream should first resolve source context.

Then the Interpreter executes the complete program.

Initial implementation may focus only on stdout.

Example:

```text
numbers = [1, 2, 3, 4]

for each even number:
    print number * 2
```

Output:

```text
4
8
```

---

# 63. Interpreter Prompt Principles

The Interpreter should be instructed to:

- act as the runtime for Dream;
    
- understand ordinary programming conventions;
    
- execute the supplied pseudocode;
    
- use imported Dream source as part of the program;
    
- resolve small ambiguities in normal mode;
    
- reject important ambiguities in strict mode;
    
- avoid explanations;
    
- avoid Markdown;
    
- return only program-visible output;
    
- avoid fabricating real-world effects.
    

---

# 64. Interpreter Effects

A pure LLM cannot truthfully perform arbitrary external operations.

Dream:

```text
read users.json
```

cannot be executed correctly unless Dream actually supplies the file contents.

Likewise:

```text
make an HTTP request
```

should not be hallucinated.

Future `dream now` should expose controlled runtime tools.

---

# 65. Future Runtime Capabilities

Conceptually:

```text
Interpreter
    │
    ├── read_file
    ├── write_file
    ├── read_stdin
    ├── write_stdout
    ├── get_env
    └── http_request
```

The Interpreter requests an effect.

Dream performs the real operation.

Dream returns the result.

This changes the interpreter from:

```text
LLM pretends to run code
```

to:

```text
LLM controls a bounded runtime
```

---

# 66. Capability Ownership

Core rule:

> **Dream owns capabilities. The model owns interpretation.**

Dream controls:

- filesystem access;
    
- source access;
    
- output directories;
    
- process execution;
    
- environment variables;
    
- build invocation;
    
- network access;
    
- secrets;
    
- runtime effects.
    

The model determines:

- source meaning;
    
- implementation decisions;
    
- target-native structure;
    
- dependency choices;
    
- project composition.
    

---

# 67. Security

Important rules:

- LLM output is untrusted.
    
- Imports must remain within allowed project boundaries.
    
- Source-reading tools must validate paths.
    
- Generated shell commands must not be executed blindly.
    
- Model-provided build metadata must be validated.
    
- The OpenAI API key must never be embedded in generated target projects.
    
- Secrets should not automatically propagate to generated processes.
    
- Strict mode is not a sandbox.
    
- Successful compilation does not imply generated software is safe.
    
- Runtime effects should be capability-gated.
    
- Output directories should be handled carefully to avoid unwanted overwrite behavior.
    

---

# 68. Dream Errors

Basic format:

```text
DreamError: ...
```

Examples:

```text
DreamError: ambiguous ordering for User.
```

```text
DreamError: imported source `./math.foo` does not exist.
```

```text
DreamError: import escapes the project root.
```

```text
DreamError: target project composition failed.
```

```text
DreamError: generated Rust project failed to build.
```

Specific error types can be introduced later.

---

# 69. Configuration

Initial configuration should come from `.env`.

Example:

```env
OPENAI_API_KEY=...
DREAM_MODEL=...
```

Potential future settings:

```env
DREAM_DEFAULT_TARGET=rust
```

Do not build a general provider abstraction until one is actually needed.

OpenAI-only is fine initially.

---

# 70. Default Target

Dream may eventually support:

```env
DREAM_DEFAULT_TARGET=rust
```

Then:

```bash
dream app.foo
```

would behave like:

```bash
dream app.foo -t rust
```

If no default target exists, Dream can require `--target`.

This remains an implementation decision for the first version.

---

# 71. Suggested CLI

Core:

```text
dream [OPTIONS] <FILE>
dream now [OPTIONS] <FILE>
```

Examples:

```bash
dream app.foo -t rust
dream app.foo -t rust -o ./out
dream app.foo -t go -o ./out
dream app.foo -t python -o ./out --run

dream --strict app.foo -t rust

dream now app.foo
dream now --strict app.foo
```

Future:

```bash
dream app.foo -t rust --run
dream app.foo -t rust --release
dream app.foo -t rust --no-build
```

---

# 72. Suggested Initial Rust Layout

Keep the first version small:

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

Do not create an elaborate module tree before functionality requires it.

---

# 73. Later Rust Layout

As Dream grows:

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
    │
    ├── llm/
    │   ├── mod.rs
    │   └── openai.rs
    │
    ├── source/
    │   ├── mod.rs
    │   ├── project.rs
    │   ├── import.rs
    │   └── resolver.rs
    │
    ├── interpreter/
    │   ├── mod.rs
    │   └── prompt.rs
    │
    ├── composer/
    │   ├── mod.rs
    │   ├── agent.rs
    │   ├── prompt.rs
    │   ├── project.rs
    │   └── repair.rs
    │
    ├── builder/
    │   ├── mod.rs
    │   ├── known.rs
    │   └── manifest.rs
    │
    └── runner/
        └── mod.rs
```

Only create these boundaries once they are useful.

---

# 74. Testing Strategy

Dream needs multiple layers of testing.

## Deterministic Tests

Test ordinary Rust behavior:

- CLI parsing;
    
- project root detection;
    
- import path validation;
    
- source resolution;
    
- cycle detection;
    
- output-directory handling;
    
- Builder process invocation;
    
- Runner behavior;
    
- error formatting.
    

## Model Behavior Tests

Test whether common pseudocode produces sensible semantics or target projects.

## End-to-End Tests

Example:

```text
tests/programs/hello.foo
```

Run:

```bash
dream hello.foo -t rust -o ./tmp/hello
```

Then build/run and verify output.

---

# 75. Cross-Target Testing

Eventually a single Dream program can be tested against multiple targets.

Example:

```text
hello.foo
```

Compose as:

```text
Rust
Go
Python
```

Run each project.

Verify externally observable behavior is equivalent.

This could become one of Dream's most important semantic regression tests.

---

# 76. Prompt Versioning

Dream's Composer and Interpreter prompts affect language behavior.

Eventually prompts should be internally versioned.

Conceptually:

```text
composer prompt v4
interpreter prompt v2
```

This becomes especially important if semantic locking or caching is added.

Not required for the first prototype.

---

# 77. Caching

Dream should eventually cache model work where appropriate.

Potential cache inputs:

```text
resolved source graph
target
strictness
model
prompt version
composition policy
```

This becomes valuable for large projects.

Not required initially.

---

# 78. Cost Model

`dream now` uses the model each time:

```bash
dream now app.foo
```

Normal Dream primarily uses the model during composition:

```bash
dream app.foo -t rust
```

After that:

```bash
cd out
cargo run
```

does not involve Dream or an API call.

This is a major advantage of normal Dream mode.

---

# 79. Non-Goals for Early Dream

Do not initially build:

- a conventional parser;
    
- a complete grammar;
    
- a custom VM;
    
- machine-code generation;
    
- a giant semantic IR;
    
- an optimizer;
    
- a package manager;
    
- a plugin system;
    
- a custom build system;
    
- a fixed list of target languages;
    
- custom dependency management for every ecosystem.
    

Dream should aggressively reuse conventional languages and toolchains.

---

# 80. MVP Phase 1 — Interpreter

Implement:

```bash
dream now main.foo
dream now --strict main.foo
```

Requirements:

- CLI parsing;
    
- `.env`;
    
- OpenAI API key;
    
- configured model;
    
- source loading;
    
- interpreter prompt;
    
- strict interpreter prompt;
    
- stdout;
    
- useful API errors.
    

Single-file support is acceptable for the first spike.

Goal:

> Prove that executable pseudocode is fun and useful enough to continue.

---

# 81. MVP Phase 2 — Rust Composition

Implement:

```bash
dream main.foo -t rust -o ./out
```

Pipeline:

```text
Dream source
    ↓
Composer
    ↓
Cargo project
    ↓
Builder
    ↓
cargo build
```

Initial Dream program can be tiny:

```text
x = 5
y = 10
print x + y
```

Success criterion:

The generated Rust project builds and produces:

```text
15
```

---

# 82. MVP Phase 3 — Run Support

Implement:

```bash
dream main.foo -t rust -o ./out --run
```

Dream:

1. composes the Rust project;
    
2. builds it;
    
3. locates the resulting artifact;
    
4. runs it;
    
5. forwards stdin/stdout/stderr.
    

---

# 83. MVP Phase 4 — Multi-File Dream

Add:

```text
import "./other.foo"
```

Implement:

- project root;
    
- recursive import resolution;
    
- cycle detection;
    
- constrained source access;
    
- source dependency graph;
    
- shared resolution for both Composer and Interpreter.
    

---

# 84. MVP Phase 5 — Arbitrary Targets

Allow arbitrary target strings:

```bash
dream main.foo -t python
dream main.foo -t go
dream main.foo -t "Common Lisp"
dream main.foo -t brainfuck
```

Do not require target registration.

The Composer receives the target as context.

---

# 85. MVP Phase 6 — Builder Support

Add Builder support for common ecosystems:

- Rust;
    
- Go;
    
- C;
    
- C++;
    
- Python;
    
- JavaScript;
    
- TypeScript;
    
- Java;
    
- Kotlin.
    

Unknown targets may still be composed successfully even if Dream cannot build them.

---

# 86. MVP Phase 7 — Bounded Repair

Add:

```text
compose
  ↓
build/check
  ↓
diagnostics
  ↓
Composer repair
  ↓
build/check
```

Use a small explicit maximum retry count.

---

# 87. Later Phase — Runtime Effects

Give `dream now` controlled capabilities for:

- files;
    
- stdin;
    
- environment;
    
- HTTP;
    
- other explicitly approved effects.
    

Keep runtime capability enforcement outside the LLM.

---

# 88. Later Phase — Semantic IR

Only introduce a formal Dream IR once direct composition causes real problems.

Potential future architecture:

```text
Dream Project
    ↓
Source Resolver
    ↓
Semantic Frontend
    ↓
Dream IR
    ├── Composer: Rust
    ├── Composer: Go
    ├── Composer: Python
    └── ...
```

At that stage, the word **compiler** may become appropriate for some deterministic internal stages.

---

# 89. Later Phase — Semantic Lock

Allow:

```bash
dream lock main.foo
```

to persist accepted semantics.

Then Dream can create multiple targets from one fixed meaning.

---

# 90. Example Multi-File Project

```text
calculator/
├── main.foo
└── math.foo
```

`math.foo`:

```text
function double x:
    return x * 2
```

`main.foo`:

```text
import "./math.foo"

number = 21

print double number
```

Compose as Rust:

```bash
dream main.foo -t rust -o ./out
```

Possible generated project:

```text
out/
├── Cargo.toml
└── src/
    ├── main.rs
    └── math.rs
```

Program output:

```text
42
```

---

# 91. Same Dream, Different Target

```bash
dream main.foo -t python -o ./python-out
```

Possible project:

```text
python-out/
├── pyproject.toml
└── calculator/
    ├── __main__.py
    └── math.py
```

Observable behavior should remain equivalent.

---

# 92. Same Dream, Immediate Execution

```bash
dream now main.foo
```

The Source Resolver gathers `math.foo`.

The Interpreter receives the complete required context.

Output:

```text
42
```

No target language is involved.

---

# 93. Strict Example

```text
import "./users.foo"

users = load users

sort them

print the first one
```

Normal mode may infer an ordering.

Strict mode:

```bash
dream --strict app.foo -t rust
```

should return something like:

```text
DreamError: `sort them` does not specify an ordering for User.
```

---

# 94. Product Positioning

Dream should be described as:

> **Executable pseudocode.**

Not:

> AI builds your app for you.

The distinction matters.

Dream is a programming language.

The programmer expresses program structure and intent.

Dream resolves that notation into executable behavior.

---

# 95. Branding

The commands themselves support the product identity.

```bash
dream foo.foo
```

Turn a Dream into real software.

```bash
dream now foo.foo
```

Run the Dream immediately.

The project can be playful without making its technical model unclear.

---

# 96. Core Architectural Rule

> **Dream owns capabilities. The model owns interpretation and composition.**

---

# 97. Core Composition Rule

> **The Composer produces a complete conventional project. It does not own the build process.**

---

# 98. Core Build Rule

> **The Builder operates on the completed target project using controlled local toolchains.**

---

# 99. Core Multi-File Rule

> **Dream gathers all required source context before interpretation or composition begins.**

---

# 100. Core Target Rule

> **Targets are open-ended. Builder support is a convenience, not a restriction.**

---

# 101. Core Strictness Rule

> **Normal Dream resolves reasonable ambiguity. Strict Dream exposes it.**

---

# 102. Core Artifact Rule

> **A generated target project is a first-class output of Dream and should remain useful without Dream.**

---

# 103. Initial Architecture

```text
                         Dream Project
                              │
                              ▼
                       Source Resolver
                              │
                              ▼
                    Resolved Source Graph
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

# 104. Possible Long-Term Architecture

```text
                         Dream Project
                              │
                              ▼
                       Source Resolver
                              │
                              ▼
                     Semantic Frontend
                              │
                              ▼
                           Dream IR
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
       Rust Composer      Go Composer    Python Composer
             │                │                │
             ▼                ▼                ▼
       Rust Project       Go Project      Python Project
             │                │                │
             ▼                ▼                ▼
           Builder          Builder          Builder
```

---

# 105. Final Design Summary

Dream source:

```text
import "./users.foo"

active users = users that aren't disabled

sort them oldest first

print the first five names
```

Run immediately:

```bash
dream now main.foo
```

Compose as Rust:

```bash
dream main.foo -t rust -o ./out
```

Compose as Python:

```bash
dream main.foo -t python -o ./out
```

Compose as anything else:

```bash
dream main.foo -t "whatever language makes sense for a TI-84 Plus CE" -o ./out
```

The defining idea is:

> **The programmer writes the intent as pseudocode. Dream resolves the program, composes it into executable software, and lets conventional tools take over from there.**