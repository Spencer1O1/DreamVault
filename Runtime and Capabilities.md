**Status:** Preliminary  
**Purpose:** Define runtime effects, tool families, and Dream's security principle.

---

## Dream Owns Every Effect

The model asks. Dream performs.

```text
LLM pretends to execute
```

is wrong.

```text
LLM requests a capability
    ↓
Dream does it
    ↓
result goes back to the LLM
```

is the architecture.

Chat text is discarded. It is not program output.

## Three Tool Families

Do not give one `write_file` to every mode.

### Source — both `now` and compose

```text
list_source_files
read_source_file(path)
```

A programmer can look at the folder. The interpreter can too.

`list_source_files` returns `.foo` paths under the project root. Paths only.

`read_source_file` returns one semantic unit, sandboxed to the project, and records the dependency. Always the `.foo` text. In compose mode, an unlocked unsettled unit is composed first; a read that already has a realization includes those artifacts. `dream now` does not compose and does not attach artifacts. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

### Interpreter runtime — `dream now` only

v0:

```text
stdout(text)
stdin()
```

`stdout` writes to real stdout immediately. Multiple calls are the print stream, in order.

If the program's meaning is to produce a result (`return the first five`), send that through `stdout`.

`stdin` reads real stdin. It may block. EOF is fine in non-interactive use.

Later, same family:

```text
read_file
write_file
http_request
```

Still executed by Dream. Still sandboxed. Not in v0.

If a v0 program needs a data file or the network, `DreamError`. Do not invent `users.json`.

### Composer — default mode only

```text
write_output_file(path, contents)
```

**v0:** writes into the `-o` staging tree. After the compose loop settles, Dream replaces the output folder.

**Next:** writes in place, only for the current unit’s artifacts. Project-owned paths use project tools. Unmanaged paths are rejected. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

The Composer does not run the program. It does not get `stdout`, `stdin`, or data-file tools.

The Interpreter does not get `write_output_file`.

## Source Requests Are Not Runtime File Access

These are separate:

```text
read_source_file("users/load.foo")
```

and later:

```text
read_file("users.json")
```

The first is another semantic unit.

The second is the program reading data at runtime.

Do not conflate them.

## Security Principle

> **Dream owns capabilities. The model owns interpretation and composition.**

Dream controls:

- source listing and reading;
- filesystem boundaries;
- target output directories;
- build execution;
- process execution;
- environment variables;
- secrets;
- runtime tools.

The model should never receive unrestricted local-machine authority.

## Related

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Core Rules|Core Rules]]
