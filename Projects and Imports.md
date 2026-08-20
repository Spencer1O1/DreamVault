**Status:** Preliminary  
**Purpose:** Define Dream project structure and how one `.foo` file uses another.

---

## Project Structure

A Dream project is a directory of semantic units.

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

Each `.foo` file is independently addressable by its project-relative path.

There is no import grammar.

## How Files Find Each Other

If `main.foo` says to use the active-users behavior, the model may request:

```text
users/active.foo
```

Dream serves that file if it is inside the project.

The programmer can mention another unit however they want:

```text
use users/active.foo
```

```text
filter with the active-users thing
```

```text
import "./users/active.foo"
```

Those are all just meaning.

The interpreter decides that another semantic unit is needed and asks Dream for it.

It may first look at the project the way a programmer looks at the folder.

## Source Access Tools

```text
list_source_files
read_source_file(path)
```

`list_source_files` returns every project-relative `.foo` path. No contents. Compose also marks whether each unit is locked. The model should list instead of inventing filenames.

`read_source_file`:

- resolves the path against the project root;
- rejects anything outside the project;
- returns only that `.foo` source;
- records that the current unit now depends on that path.

The model does not get arbitrary filesystem access.

Requesting `users/active.foo` is source resolution.

Reading `users.json` at runtime is program execution.

Do not conflate them. See [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]].

## The Graph Is an Output

The first time a unit is dreamed, Dream does not already know its dependencies.

```text
interpret or compose a .foo unit
    ↓
model requests other .foo files
    ↓
Dream serves them if allowed
    ↓
record the discovered dependency set
```

That recorded set *is* the source graph for later runs.

If the unit's source has not changed, Dream can reuse the accepted meaning and the recorded dependencies without asking the model which files it needs.

This is the same move as file-level units:

- the filesystem gives unit identity;
- the model discovers relationships;
- the cache remembers them until the source changes.

## What Stays Mechanical

```text
DETERMINISTIC
────────────────────────
project root
file paths
.foo unit identity
path sandbox
recorded dependency set after a dream
configuration

FUZZY
────────────────────────
meaning of each .foo file
which other units a file needs, until that has been recorded
```

Dream should not invent an `import` keyword just to make the first graph cheap.

## Missing Files and Cycles

If the model requests a file that does not exist, Dream reports that.

If A requests B and B requests A, that is two reads, not a process error. Re-reading a unit (including the entry) is fine. The session records a set of units that were read. A request stack that treated last-read as the parent used to abort compose when the model re-read the entry for its artifacts.

These checks happen while units are being dreamed, not by parsing a formal prelude.

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

A manifest is not required for the earliest prototype.

## Related

- [[Projects/Dream/MVP|MVP]]
- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Architecture|Architecture]]
- [[Projects/Dream/Language and Source|Language and Source]]
- [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]]
