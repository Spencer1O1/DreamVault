**Status:** Historical v0  
**Purpose:** The first CLI that was built (replace `-o`, builder after writes). Do not treat this as the current compose contract. Current compose is [[Projects/Dream/Artifact Ownership|Artifact Ownership]] through Phase 8. Interpreter behavior in this file still matches the crate (`--lucid`; v0 used a `now` subcommand). Plus [[Projects/Dream/Core Rules|Core Rules]].

The Rust crate lives in a **separate workspace**, not this notes vault.

---

## CLI

```bash
dream [--lucid] [--strict] <file.foo>
dream [--strict] [--no-warn] <file.foo> -t <target> -o <dir>
dream [--strict] [--no-warn] <file.foo> -t <target> -o <dir> --build
dream [--strict] [--no-warn] <file.foo> -t <target> -o <dir> --run
```

- `--lucid` interprets. Observable output is whatever the model sends through `stdout`.
- Default `dream` **composes only**. It does not build or run.
- `--build` composes, then builds if Dream knows a toolchain for the declared builder.
- `--run` composes, builds, and runs.
- `--strict` is a stricter prompt. It is not a parser or linter.
- `--no-warn` treats toolchain warnings as a failed build. It is compose-only.
- `-t` is an open-ended string.
- `-o` is required for compose. **v0 replaces the whole folder.** Stage the new tree, then swap, so a failed compose does not half-wipe the destination.

v0 config:

```env
OPENAI_API_KEY=...
DREAM_MODEL=...
```

No provider abstraction.

## Tool Families

Dream owns every capability. The model only asks. Chat text is discarded.

### Source — both modes

`list_source_files`

- returns project-relative paths of every `.foo` file under the project root
- paths only, no file contents

`read_source_file(path)`

- returns that `.foo` source if it is inside the project
- reject anything else
- record that the current unit depends on that path

A programmer can look at the folder. The model may do the same. It must not guess filenames without listing, and it must not require an `import` keyword.

### Interpreter runtime — `--lucid` only

`stdout(text)`

- Dream writes `text` to real stdout immediately
- multiple calls are the program's print stream, in order
- if the program's meaning is to produce a result (`return the first five`), send that result through `stdout`

`stdin()`

- Dream reads from real stdin and returns it
- may block; EOF in non-interactive use is fine

v0 has no data-file or network tools. If the program needs `users.json` or HTTP, `DreamError`. Do not invent file contents.

Later, same family: `read_file`, `write_file`, `http_request`. Still executed by Dream. Still sandboxed.

### Composer — default mode only

`write_output_file(path, contents)`

- **v0:** writes into a staging directory for `-o`
- path must stay under the output root
- after the compose loop settles, Dream replaces `-o` with the staged tree

**v0 asked `set_builder` after the write loop.** The crate now declares the builder first and writes in place. See [[Projects/Dream/Artifact Ownership|Artifact Ownership]].

The Composer does not get `stdout`, `stdin`, or data-file tools. It is writing a project, not running the program.

The Interpreter does not get `write_output_file`.

## Settle Rule

A run is finished when the model stops calling tools, or hits a turn cap.

Do not treat a unit as complete, or (in v0) swap `-o`, until source requests have settled.

Record the requested `.foo` paths. That is the source graph for this run.

## Errors

```text
DreamError: ...
```

At least:

- requested source does not exist
- source request escapes project root
- output write escapes `-o`
- (historical) cycle in source requests — a request-stack abort; do not treat a re-read as a cycle
- program needs a runtime capability v0 does not have
- `--strict` and the model is too unsure to continue
- build or run failed

## Prompts

Keep them short.

Interpreter: you execute this Dream program. Request source instead of inventing other `.foo` files. Send all observable results through `stdout`. Do not chat. Do not invent I/O.

Composer: you write a complete, hand-maintainable target project under `-o`. Request source instead of inventing other `.foo` files. Write files only through `write_output_file`. Do not execute the program. Do not chat.

`--strict` adds: do not guess important semantics; return a `DreamError` instead.

## Out of Scope for v0

These were not built in v0. Several are now specified for the next work:

- in-place reconcile, provenance, `--fresh` — [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- semantic lock / inspect — no `redream` command; normal `dream` reconciles
- `dream.toml`
- data-file / HTTP tools
- a formal IR or Gimbal
- provider plugins

v0 **did** add bounded build repair (`DREAM_REPAIR_CAP`) and `--no-warn`. Those stay.

## Suggested First Examples

1. Single-file `now` that prints something.
2. Two-file `now` where the entry mentions another unit without an import keyword.
3. Tiny compose-to-Rust with `-o`, no `--build`.

## Related

- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/Implementation Plan|Implementation Plan]]
- [[Projects/Dream/Core Rules|Core Rules]]
