**Status:** Preliminary  
**Purpose:** Define target-aware composition, open-ended targets, builders, and repair.

---

## Target-Aware Composition

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

## Arbitrary Targets

Generation should remain open-ended.

Dream should not reject:

```bash
dream main.foo -t cobol
```

simply because COBOL does not have a built-in adapter.

If the Composer can generate it, generation may succeed.

## Known Builders

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

## Generated Build Metadata

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

## Composition and Building Stay Separate

Core invariant:

> **The Composer writes the target project. The Builder invokes toolchains.**

The Composer should not receive unrestricted shell access.

It writes only through `write_output_file`, into a staging directory for `-o`. After the compose loop settles, Dream replaces the output folder.

The Composer does not get `stdout`, `stdin`, or data-file tools. Those belong to `dream now`.

Default compose does not build. See `--build` and `--run` in [[Projects/Dream/CLI and Execution|CLI and Execution]].

## Build Repair

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

## Generated Project Is Not the Semantic IR

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

A later architecture may introduce a formal semantic representation before target generation. That is not decided yet. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

## Related

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Core Rules|Core Rules]]
