**Status:** Preliminary  
**Purpose:** Define Dream project structure, imports, and the deterministic source graph.

---

## Project Structure

A Dream project is fundamentally a graph of semantic units.

Example:

```text
my-project/
├── main.foo
├── models/
│   └── user.foo
├── users/
│   ├── load.foo
│   ├── active.foo
│   └── oldest.foo
└── dream.toml
```

Each `.foo` file is independently addressable.

Imports define relationships between those units.

## Deterministic vs Fuzzy Structure

Dream deliberately places deterministic structure around fuzzy semantics.

```text
DETERMINISTIC
────────────────────────
project root
file paths
.foo unit identity
imports
dependency graph
configuration

FUZZY
────────────────────────
meaning of each .foo file
```

The source graph exists independently of the LLM.

It should be possible to construct and inspect it mechanically.

## Imports

Imports are one of the intentionally formal parts of Dream.

Canonical syntax:

```text
import "./users/active.foo"
```

Possible alias syntax:

```text
import "./users/active.foo" as active_users
```

Exact namespace rules may evolve, but import paths should remain structurally deterministic.

Imports operate on whole semantic units.

## Import Semantics

If:

```text
main.foo
```

contains:

```text
import "./users/active.foo"
```

then `main.foo` depends on the semantic unit represented by:

```text
users/active.foo
```

This relationship is part of Dream's source dependency graph.

Dream should never need an LLM to determine whether the import exists or what file it refers to.

## Source Dependency Graph

Example:

```text
main.foo
├── users/load.foo
├── users/active.foo
│   └── models/user.foo
└── users/oldest.foo
```

The graph is deterministic.

The semantics inside each node may initially require an LLM.

This distinction is central to Dream.

## Project Root

Dream should identify a project root.

Eventually projects may use:

```text
dream.toml
```

Example:

```toml
[project]
name = "my-project"
entry = "main.foo"
```

Potential future project settings:

```toml
[project]
name = "my-project"
entry = "main.foo"
default_target = "rust"
```

A manifest is not required for the earliest prototype.

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
