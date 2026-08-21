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

Same names are fine. The implementations are not. Interpreter `read_file` / `write_file` are data files. Composer `read_file` / `write_file` are dest. Do not share one implementation across modes.

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

Same family, current:

```text
list_files
read_file
write_file
http_request
```

Still executed by Dream. Still sandboxed to the project root. Not composer writes. Not `.foo` files (`read_source_file` is the unit). Not `.dream/`. `http_request` is the **program** reaching the network, not the composer researching docs. Official language docs are the catalog `docs` URL on `set_toolchain`. Composer fetch (MCP / indexes) is later. See [[Projects/Dream/Later Composer Tools|Later Composer Tools]].

### Composer — default mode only

```text
read_file(path)
write_file(unit, path, contents)
remove_file(unit, path)
```

Writes in place. `unit` is the project-relative `.foo` that owns the path, except this toolchain’s setup files (no unit). Dream checks the claim. Setup paths on the catalog row are project-owned and composer-writable. Lockfiles and build dirs are not. Unmanaged paths are rejected. `write_file` is a whole-file replace. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

Later, same composer side: research tools (indexes, docs) and targeted / LSP edits. Not built. Not interpreter runtime. See [[Projects/Dream/Later Composer Tools|Later Composer Tools]].

The Composer does not run the program. It does not get `stdout`, `stdin`, or data-file tools.

The Interpreter does not get composer dest writes.

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
