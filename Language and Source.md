**Status:** Preliminary  
**Purpose:** Define Dream source files and what is *not* a syntax.

---

## Source Extension

Dream source files use:

```text
.foo
```

Examples:

```text
main.foo
load_users.foo
server.foo
user.foo
```

The extension is intentionally unrelated to the language name.

Keep it.

## People Write Whatever They Want

A `.foo` file has no grammar.

The programmer can write ordinary pseudocode, mixed notation, prose, diagrams in text, notes, or anything else that conveys the unit's meaning.

Dream interprets that meaning.

It does not lock the programmer into a comment style, statement style, or official notation inside the file.

These are all valid source, if they describe one semantic unit:

```text
users = load users

for user in users:
    if user is active:
        print user.name
```

```text
take all active users

print each one's name
```

```text
only the visible accounts — ignore deactivated ones
```

Notes and asides are just part of the file.

There is no comment syntax.

`//`, `#`, `/* */`, parentheses, or a sentence in English are all just text the interpreter reads with everything else.

## What Documentation Shows

Dream docs may show readable pseudocode such as:

```text
load the users
keep the active ones

for user in users:
    print user.name
```

That is an example, not a required style.

Using another `.foo` file is the same: the programmer mentions it however they want, and the model requests the file. See [[Projects/Dream/Projects and Imports|Projects and Imports]].

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/Vision and Principles|Vision and Principles]]
