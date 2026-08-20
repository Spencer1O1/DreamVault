**Status:** Preliminary  
**Purpose:** Define how Dream handles ambiguity in normal and strict modes.

---

## Strict Mode

Both modes support:

```bash
--strict
```

Examples:

```bash
dream --strict app.foo -t rust
```

```bash
dream now --strict app.foo
```

Strict mode changes semantic inference.

It does not introduce a separate source grammar.

## Default Ambiguity Handling

Normal Dream should resolve reasonable ambiguity.

Example:

```text
sort the users
```

If context strongly implies a conventional ordering, Dream may choose it.

Normal Dream should not demand specification of every obvious detail.

## Strict Ambiguity Handling

Strict Dream should reject important ambiguity.

Example:

```text
sort the users
```

Possible result:

```text
DreamError: ambiguous ordering for User.
Specify a field or ordering criterion.
```

Strict mode should avoid:

- creative guesses;
- invented data;
- invented effects;
- unjustified assumptions.

## Assumptions Become Part of Meaning

Normal mode may allow reasonable assumptions.

Those assumptions should eventually be recorded in semantic metadata.

Example:

```text
"sort users"
    ↓
assumed ordering: name ascending
```

If accepted, that assumption becomes part of the semantic result rather than being re-guessed every compilation.

A semantic result accepted under normal mode may not automatically satisfy strict mode.

Cache keys should therefore include semantic policy such as:

```text
strictness
frontend version
```

A normal-mode semantic result should not be silently reused as a strict-mode result unless Dream can establish that it satisfies strict requirements.

## Related

- [[Projects/Dream/CLI and Execution|CLI and Execution]]
- [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]]
- [[Projects/Dream/Core Rules|Core Rules]]
