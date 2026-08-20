**Status:** Preliminary  
**Purpose:** Define Dream source files, recommended notation, and comments.

---

## Source Extension

Canonical Dream source files use:

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

## Dream Source Style

Dream should encourage pseudocode that still looks like programming.

Good:

```text
users = load users

for user in users:
    if user is active:
        print user.name
```

Also good:

```text
take all active users

print each one's name
```

Less ideal:

```text
Please make me an app that loads some users and displays them somehow.
```

Dream may understand prose-heavy source, but the language should encourage clear program intent rather than generic prompting.

## Recommended Style

Documentation should prefer readable pseudocode such as:

```text
import "./users/active.foo"

users = load users

users = active(users)

for user in users:
    print user.name
```

Recommended syntax is a convention.

It does not imply a complete parser grammar.

## Comments

Canonical comments use:

```text
// comment
```

Example:

```text
// only return accounts visible to the current user

return visible accounts
```

Other styles may sometimes be understood, but `//` is the official form.

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Projects and Imports|Projects and Imports]]
- [[Projects/Dream/Vision and Principles|Vision and Principles]]
