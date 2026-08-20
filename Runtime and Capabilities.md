**Status:** Preliminary  
**Purpose:** Define runtime effects, the capability model, and Dream's security principle.

---

## Runtime Effects

A pure LLM interpreter cannot truthfully perform arbitrary external operations.

Dream source such as:

```text
read users.json
```

should not cause the model to invent file contents.

Future `dream now` should expose controlled runtime capabilities.

## Runtime Capability Model

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

## Source Requests Are Not Runtime File Access

These are separate:

```text
read_source_file("users/load.foo")
```

and:

```text
read users.json
```

The first is the interpreter asking Dream for another semantic unit.

The second is the program reading data at runtime.

Do not conflate them.

## Security Principle

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

## Related

- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Core Rules|Core Rules]]
