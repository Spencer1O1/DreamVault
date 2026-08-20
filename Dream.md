**Status:** Preliminary specification  
**Name:** Dream  
**Current stage:** Compose is [[Projects/Dream/Artifact Ownership|Artifact Ownership]] through Phase 9 (in-place, one session, writes name the unit, Dream owns manifests). `--lucid` still matches [[Projects/Dream/MVP|MVP]] interpreter behavior. Next: Phase 10 locks.

Dream is a programming language for executable pseudocode.

A Dream program is written in `.foo` files using clear, informal programming notation rather than a rigid grammar. That source is **foocode**.

Dream uses an LLM to determine what each source unit means, then either composes that program into a conventional software project in a requested target language, or executes it immediately.

The core idea is:

> **Dream is executable pseudocode.**

Dream is not primarily an AI coding assistant.

The programmer writes the program.

Dream interprets the programmer's notation.

The most important structural rule is:

> **One `.foo` file is one semantic unit.**

## Project Documents

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Vision and Principles|Vision and Principles]]
- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Language and Source|Language and Source]]
- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Semantics and Strictness|Semantics and Strictness]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
- [[Projects/Dream/Targets and Composition|Targets and Composition]]
- [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]]
- [[Projects/Dream/Semantic Locking and Inspection|Semantic Locking and Inspection]]
- [[Projects/Dream/Implementation Plan|Implementation Plan]]
- [[Projects/Dream/Core Rules|Core Rules]]
- [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]]

## Two Modes

```bash
dream program.foo
```

compose that Dream program into a conventional software project in a requested target language (build and run only if asked);

or:

```bash
dream --lucid program.foo
```

execute the Dream program immediately using an LLM as the interpreter.

## Preliminary Architecture

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
         `--lucid`                    default
        Interpreter                 set_builder
         stdout/stdin                     │
              │                           ▼
       Program Output            per-unit reconcile
                                      --build / --run
```

The diagram is the current architecture. v0 used to replace `-o` and ask for the builder last.

Gimbal is not part of the current design. A later formal semantic core is discussed separately in [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].
