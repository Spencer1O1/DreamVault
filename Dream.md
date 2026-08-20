**Status:** Preliminary specification  
**Name:** Dream  
**Current stage:** Foundational design; implementation has not started

Dream is a programming language for executable pseudocode.

A Dream program is written in `.foo` files using clear, informal programming notation rather than a rigid grammar.

Dream uses an LLM to determine what each source unit means, then either composes that program into a conventional software project in a requested target language, or executes it immediately.

The core idea is:

> **Dream is executable pseudocode.**

Dream is not primarily an AI coding assistant.

The programmer writes the program.

Dream interprets the programmer's notation.

The most important structural rule is:

> **One `.foo` file is one semantic unit.**

## Project Documents

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

compose that Dream program into a conventional software project in a requested target language, build it when possible, and optionally run it;

or:

```bash
dream now program.foo
```

execute the Dream program immediately using an LLM as the interpreter.

## Preliminary Architecture

```text
                       Dream Project
                            │
                            ▼
                     Source Resolver
                       project root
                       path sandbox
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
           `now`                      default
        Interpreter                   Composer
              │     read_source_file      │
              └──────────►────────────────┘
                            │
                     Program Output
                     or Target Project
```

Gimbal is not part of the current design. A later formal semantic core is discussed separately in [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].
