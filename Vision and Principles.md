**Status:** Preliminary  
**Purpose:** Define what Dream is trying to become and the principles that constrain its design.

---

## Vision

**Dream** is a programming language for executable pseudocode.

A Dream program is written in `.foo` files using clear, informal programming notation rather than a rigid grammar.

Dream uses an LLM to determine what each source unit means, then either:

```bash
dream program.foo
```

compose that Dream program into a conventional software project in a requested target language, build it when possible, and optionally run it;

or:

```bash
dream now program.foo
```

execute the Dream program immediately using an LLM as the interpreter.

The core idea is:

> **Dream is executable pseudocode.**

Dream is not primarily an AI coding assistant.

The programmer writes the program.

Dream interprets the programmer's notation.

## What Dream Is For

Dream should let a programmer write a program in the notation they would use to explain the program, then either run that program immediately or turn it into ordinary software in another language.

The generated project should remain useful without Dream. A developer should be able to take over maintenance manually. Dream can therefore bootstrap conventional software.

`dream now` preserves the original Dream idea:

```text
pseudocode
    ↓
LLM
    ↓
program output
```

It is useful for:

- experiments;
- tiny programs;
- prototypes;
- learning;
- quick scripts;
- testing Dream semantics.

It is intentionally less deterministic than normal composition.

Normal Dream and `dream now` therefore serve distinct purposes.

## What Dream Is Not

Dream is not a prompt-to-app generator.

The programmer still writes the program.

There is no required notation inside a `.foo` file.

Dream is also not, in the current design, a formal language with a complete parser grammar.

## Deterministic Structure Around Fuzzy Meaning

Dream deliberately places deterministic structure around fuzzy semantics.

```text
DETERMINISTIC
────────────────────────
project root
file paths
.foo unit identity
path sandbox
recorded dependencies after a dream
configuration

FUZZY
────────────────────────
meaning of each .foo file
which other units a file needs, until that has been recorded
```

After semantic interpretation, Dream should eventually return to a deterministic representation:

```text
.foo source
    ↓
semantic interpretation
    ↓
formal semantic representation
```

What that formal representation is remains open. See [[Projects/Dream/Later Formal Semantic Core|Later Formal Semantic Core]].

## Final Definition

> **Dream is an executable pseudocode language where each `.foo` file is an independently meaningful semantic unit, and projects can either run immediately through an LLM or be composed into ordinary software projects in virtually any target language.**

## Related

- [[Projects/Dream/Semantic Units|Semantic Units]]
- [[Projects/Dream/Language and Source|Language and Source]]
- [[Projects/Dream/Core Rules|Core Rules]]
