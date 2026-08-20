**Status:** Later  
**Purpose:** Persist accepted meaning for `.foo` units and make semantic state inspectable.

---

## Locking

Dream should support locking accepted semantics.

Conceptually:

```bash
dream lock users/active.foo
```

A locked semantic unit means:

> Do not reinterpret this source unit automatically unless its lock is explicitly invalidated or a required semantic dependency makes the existing meaning unusable.

Exact lock behavior must remain compatible with correctness.

A lock should never force Dream to use an invalid semantic result.

Potential commands:

```bash
dream lock users/active.foo
dream unlock users/active.foo
dream redream users/active.foo
```

Exact CLI may change.

The underlying unit remains the `.foo` file.

## File Requests Do Not Block Locking

A lock records the *result* of a dream, not a parsed import list.

After `main.foo` is dreamed, Dream already has:

```text
accepted meaning
public interface
requested files
source hash
```

Locking that unit freezes those. Later runs reuse them. The model is not asked again which files it would like.

That is why informal file requests still allow locking:

- unit identity is the path, which is mechanical;
- the first dream produces meaning and a dependency set;
- the lock persists both.

`dream lock .` means: persist every unit that has already been dreamed. Units that have never been dreamed are dreamed first, or left unlocked.

Informal dependencies make locking *more* important, not less. Without a formal `import` line, the choice “this unit uses `users/active.foo`” *is* part of accepted meaning. Locking is how that choice stops drifting.

If a requested dependency later becomes unusable (missing file, broken interface), the lock is stale. A lock must never force an invalid result.

## Project Locking

Possible:

```bash
dream lock .
```

Meaning:

> Persist the currently accepted semantic representation of all resolved `.foo` units.

This could create a stable semantic snapshot of the project.

## Semantic Lock Is Not Source Lock

The programmer can still edit:

```text
users/active.foo
```

A semantic lock records accepted meaning.

If source and lock diverge, Dream should clearly report the unit as stale rather than silently pretending they match.

## Semantic States

Possible internal states:

```text
Missing
Valid
Locked
Stale
Invalid
```

Example:

```text
users/active.foo    Valid
users/oldest.foo    Stale
users/save.foo      Locked
```

Exact state design can evolve.

## Inspecting Semantic State

Future:

```bash
dream inspect users/active.foo
```

could show:

```text
Unit:
  users/active.foo

Source:
  unchanged

Semantic state:
  locked

Interface:
  List<User> → List<User>

Dependencies:
  models/user.foo

Target-independent:
  yes
```

## Project Inspection

Possible:

```bash
dream inspect .
```

Output could summarize:

```text
main.foo
  valid

users/load.foo
  valid

users/active.foo
  locked

users/oldest.foo
  stale
  source changed

dashboard/render.foo
  valid
  depends on users/oldest.foo
```

## Semantic Graph Inspection

Possible future command:

```bash
dream inspect . --semantic-graph
```

Conceptual graph:

```text
main.foo
├── users/load.foo
├── users/active.foo
└── users/oldest.foo
    └── models/user.foo
```

with metadata showing semantic status.

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]]
- [[Projects/Dream/CLI and Execution|CLI and Execution]]
