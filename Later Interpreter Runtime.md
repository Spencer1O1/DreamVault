**Status:** Deferred  
**Purpose:** Capture a candidate long-term shape for `dream now`. This is not part of the current interpreter. Do not build it yet.

---

## Current `dream now`

The crate already does this:

```text
.foo source
    ↓
LLM tool loop
    ↓
Dream executes each requested effect
    ↓
results go back to the LLM
    ↓
program output
```

One model turn may include several tool calls. Dream runs them in order. Their arguments were bound before any of those results existed. That is not an execution plan. Dependent calls still need another turn, or a later plan with references.

That is a working prototype. It is not the intended long-term runtime.

A later formal core is a separate question. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

## Intended Direction

> **The LLM decides what the program means. Dream decides whether that meaning is valid. Once accepted, Dream executes it without semantic improvisation.**

```text
.foo units
    ↓
LLM semantic elaboration
    ↓
candidate execution plan
    ↓
Dream validator
    ├── invalid → bounded repair → LLM again
    └── valid
         ↓
      VALIDATION BOUNDARY
         ↓
      deterministic runtime
         ↓
      program output
```

Runtime values may choose branches, drive loop counts, and fail effects. They must not send Dream back to the model to ask what the program meant.

An explicit program feature such as `ask an AI ...` is a capability, not Dream reinterpreting its own semantics.

## What Belongs Where

The LLM elaborates informal `.foo` into executable meaning.

Dream owns:

- validation;
- control flow;
- ordinary computation;
- capability-gated effects (`stdin`, `stdout`, later files and HTTP);
- runtime errors.

Tools are effects. Arithmetic, filtering, and looping should not be model round trips.

Independent effects may run concurrently once the plan names them. Dream should derive the dependency graph from references, not from a second model-produced edge list.

Do not build a generic agent workflow engine.

## Source Units

> **One `.foo` file is one semantic unit.**

Do not invent sub-file units to make a plan IR nicer.

Current law is a **request loop**: the model lists and reads units as meaning requires. See [[Projects/Dream/Architecture|Architecture]].

Do not later require every `.foo` file, or even every needed unit, to be loaded before execution or elaboration starts.

Which units matter is meaning. That meaning can depend on a runtime value (`ask which helper`, then read that `.foo`). A settle-first prelude cannot express that without either preloading unused files or forbidding the program.

A future execution plan can include `read_source_file` as an operation. Completeness still means this run's source requests have settled, not that discovery happened in a prelude.

## Errors

Keep the conceptual split for a future plan validator:

- the model could not produce valid meaning;
- a valid program hit a runtime failure;
- a runtime value violated accepted meaning (`int x` and the user typed `2.5`).

Do not lock that validator taxonomy yet.

Current host types are: `UsageError`, `ConfigError`, `InterpreterError`, `ComposerError`, `RuntimeError`. See the crate `error` module. `RuntimeError` is host plumbing (OpenAI, I/O, JSON, shared source helpers). Composition, lock, repair, and build are `ComposerError`. The dreamed program (`dream_error`, lucid turn cap) is `InterpreterError`.

`--strict` belongs to elaboration and validation, not to improvising at runtime.

## Gimbal

A future execution plan must not depend on Gimbal.

It should remain compatible with the long-term idea that `now` and compose share one accepted meaning. How that happens is undecided.

## What Not to Build Now

- a plan IR or DAG executor;
- empty `SemanticElaborator` / `Validator` / `Executor` modules;
- runtime semantic replanning;
- unbounded repair;
- a standard library of deterministic ops;
- Gimbal bindings.

The crate should keep `dream now` and `dream now --strict` as they are.

The only architectural caution: do not assume forever that every effect must return to the model before the next one can be known. The current loop is allowed. It is not the destination.

## Related

- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
- [[Projects/Dream/Implementation Plan|Implementation Plan]]
- [[Projects/Dream/MVP|MVP]]
