**Status:** Foundational  
**Purpose:** Define the semantic atom of Dream: one `.foo` file is one independently interpreted unit.

---

## Fundamental Semantic Rule

The most important structural rule in Dream is:

> **One `.foo` file is one semantic unit.**

Dream does not attempt to deterministically discover function, declaration, or semantic boundaries inside an arbitrary pseudocode file.

The filesystem provides those boundaries.

A `.foo` file is the smallest independently interpreted, cached, locked, invalidated, and re-dreamed unit.

This rule is foundational to Dream's architecture.

## Why Files Are Semantic Units

Dream intentionally allows fuzzy syntax.

That means code such as:

```text
take the active users

sort them oldest first

return the first five
```

may not contain deterministic parser-visible function boundaries.

Dream therefore should not attempt to ask:

> Where does one semantic unit end and another begin?

Instead, the programmer expresses that structure through files.

```text
users/
├── active.foo
├── oldest.foo
└── top_five.foo
```

Each file can be interpreted independently.

This gives Dream deterministic:

- unit identity;
- dependency boundaries;
- cache keys;
- lock boundaries;
- invalidation boundaries;
- re-dream boundaries.

Trying to detect internal units through the LLM would make caching and invalidation circular:

```text
need semantic units
↓
ask model where semantic units are
↓
cache semantics using boundaries produced by model
↓
source changes
↓
must ask model again to know whether boundaries changed
```

Dream avoids this entirely.

The programmer defines semantic boundaries through `.foo` files.

## Semantic Unit Kinds

A semantic unit does not necessarily mean a function.

A `.foo` file may represent:

- a function;
- a value;
- a constant;
- a type;
- a data model;
- a transformation;
- a service;
- an executable entrypoint;
- another independently meaningful program component.

The important invariant is:

> **Each `.foo` file should describe one externally meaningful semantic thing.**

Example function:

```text
users/active.foo

input is a list of users

return only the users whose active field is true
```

Example type:

```text
models/user.foo

a User has:
    name: text
    age: integer
    active: boolean
```

Example entrypoint:

```text
main.foo

load the users
keep the active ones

print each user's name
```

## Semantic Unit Identity

A semantic unit is identified primarily by its canonical project-relative path.

Example:

```text
users/active.foo
```

This identity is deterministic.

The path identifies the unit regardless of how the model understands its contents.

The path can act as the stable structural key for:

- semantic caching;
- locks;
- diagnostics;
- re-dreaming;
- dependency edges;
- source hashes;
- dependency metadata;
- invalidation.

## File Granularity Is Deliberate

Dream should not initially attempt sub-file semantic caching.

Do not introduce:

- line-level semantic IDs;
- model-detected functions inside files;
- AST-node caches;
- fuzzy internal semantic boundaries.

The `.foo` file is the semantic atom.

This can be revisited only if a deterministic source syntax eventually provides smaller stable boundaries.

## One Semantic Thing Per File

Dream should encourage:

```text
users/
├── load.foo
├── active.foo
├── oldest.foo
└── save.foo
```

rather than:

```text
users.foo
```

containing twenty unrelated functions.

This is not merely a style preference.

It improves:

- semantic caching;
- re-dreaming;
- dependency tracking;
- locking;
- model context size;
- source review;
- composability.

## Entry Files

An executable entrypoint such as:

```text
main.foo
```

is itself one semantic unit.

It may use many other units.

Its job is typically composition:

```text
load the users
keep the active ones

print each user's name
```

## Related

- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/Cache and Incremental Semantics|Cache and Incremental Semantics]]
- [[Projects/Dream/Core Rules|Core Rules]]
