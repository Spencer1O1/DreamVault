**Status:** Preliminary  
**Purpose:** Define runtime effects, tool families, and Dream's security principle.

---

## Dream Owns Every Effect

The model asks. Dream performs.

Tool descriptions describe the tool. Parameter descriptions describe the parameters. Do not explain a parameter in the tool description.

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

### Source — both `--lucid` and compose

```text
list_source_files
read_source_file(path)
```

A programmer can look at the folder. The interpreter can too.

`list_source_files` returns `.foo` paths under the project root. Paths only. Compose also marks whether each unit is locked. `--lucid` stays paths only.

`read_source_file` returns one semantic unit, sandboxed to the project, and records the dependency. Always the `.foo` text. It never starts a compose job. In compose mode, if the provenance store already has artifacts for that unit, the read attaches them. `--lucid` returns `{ path, source }` only. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

### Interpreter runtime — `--lucid` only

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

Still executed by Dream. Still sandboxed. Not in v0. `http_request` here is the **program** reaching the network, not the composer researching docs.

If a v0 program needs a data file or the network, `DreamError`. Do not invent `users.json`.

### Composer — default mode only

```text
write_output_file(unit, path, contents)
remove_output_file(unit, path)
set_dependencies(unit, dependencies)
```

Writes in place. `unit` is the project-relative `.foo` that owns the path. Dream checks the claim. `set_dependencies` is for known toolchains only; Dream owns the manifest. Project-owned paths from the toolchain must not be modified; writes and removes of those paths are rejected. Unmanaged paths are rejected. `write_output_file` is a whole-file replace. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

Later, same composer side: research tools (indexes, docs) and targeted / LSP edits. Not built. Not interpreter runtime. See [[Projects/Dream/Later Composer Tools|Later Composer Tools]].

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
