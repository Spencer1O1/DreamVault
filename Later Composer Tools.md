**Status:** Wanted. Not built.  
**Purpose:** Composer research and targeted edits. Do not start either while compose is still settling. Do not treat these as implied by optional `version` or by `write_file`.

---

Dream owns every effect. The model asks. Dream performs. Same rule as today.

These are **composer** tools. They are not `--lucid` runtime. Interpreter `http_request` (the program needs the network) is a different later family. See [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]].

## Research

The composer will need tools that reach the internet: package indexes, language docs, examples. MCP is the right later shape: Dream-owned, sandboxed fetch. Not a generic “get docs of a language” tool, and not lucid `http_request`.

Each catalog row already returns its official **docs URL** on `set_toolchain`. That is a locator, not a fetch.

That is how the model looks up an API or a current package version. It is **not** a Dream version resolver.

The composer writes versions into the setup file. Dream does not look up “current” and pin it.

Research results come back as tool output. Dream sandboxes the fetch. The model decides when to ask.

Do not add crates.io / pkg.go.dev / docs lookup as hidden compose behavior.

## Targeted edits

`write_file` stays a whole-file replace. One path, one body. Repair overwrites that body.

Later: patch or LSP-shaped tools so the model can change part of a file instead of rewriting it.

Worth it when files are large and rewrite-the-world is the pain. Not a reason to change write now.

A second mutation shape needs apply semantics (hunk, language server per target, failed-context repair). Do not grow a language-server subsystem until that is the work.

Ownership, locks, dest bounds, and unit claims stay the same. A targeted edit is still a Dream-owned write to a unit-owned path.

## Related

- [[Projects/Dream/Artifact Ownership|Artifact Ownership]]
- [[Projects/Dream/Runtime and Capabilities|Runtime and Capabilities]]
- [[Projects/Dream/Implementation Plan|Implementation Plan]]
- [[Projects/Dream/Core Rules|Core Rules]]
