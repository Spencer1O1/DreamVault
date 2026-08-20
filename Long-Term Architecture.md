Dream's long-term architecture should use **[[Gimbal]] as its formal semantic target**.

Instead of having Dream independently compose a Rust, Go, Python, or other target project, Dream should eventually translate pseudocode into a valid Gimbal program.

```text
Dream pseudocode
    ↓
semantic interpretation
    ↓
Gimbal program
    ↓
Gimbal categorical semantics
    ↓
target-language functor
    ├── Rust
    ├── Go
    ├── C
    ├── Python
    └── ...
```

This creates a clean division of responsibility:

- **Dream** determines what the programmer meant.
    
- **Gimbal** represents that program precisely.
    
- **Gimbal target translators** map the formal program into supported target languages.
    

For example:

```bash
dream app.foo -t rust
dream app.foo -t go
dream app.foo -t python
```

should eventually all begin from the same Gimbal program:

```text
app.foo
   ↓
Dream semantic frontend
   ↓
Gimbal program
   ├── F_Rust
   ├── F_Go
   └── F_Python
```

The target selection therefore happens **after semantics have been fixed**.

This is much stronger than asking an LLM to independently reinterpret the Dream program for every target.

Because Gimbal is categorically oriented, its target translators can be modeled as structure-preserving mappings from Gimbal semantics into the semantics of each target language.

Conceptually:

```text
        Gimbal
          │
          ├── F_Rust ──→ Rust
          ├── F_Go   ──→ Go
          ├── F_C    ──→ C
          └── F_...  ──→ ...
```

The exact categorical structure of these language functors will depend on Gimbal's final semantics, but the design goal is that translation should preserve the relevant program structure rather than merely regenerate equivalent-looking source code.

## Architectural Roles

### Dream

Dream remains the fuzzy, human-facing language.

Its job is:

```text
informal pseudocode
        ↓
determine programmer intent
        ↓
valid Gimbal program
```

Dream answers:

> **What did the programmer mean?**

### Gimbal

Gimbal becomes the formal semantic core.

Its job is:

```text
precise typed program
        ↓
categorical semantics
        ↓
validated program meaning
```

Gimbal answers:

> **What program is this, exactly?**

### Target Translators

Target translators take the formal Gimbal program and map it into conventional languages.

Their job is:

```text
Gimbal program
    ↓
structure-preserving translation
    ↓
target-language project
```

Target translation should ideally be deterministic and should not require another LLM interpretation step.

## Long-Term Default Pipeline

Normal Dream execution eventually becomes:

```text
Dream Project
     ↓
Source Resolver
     ↓
Dream Semantic Frontend
     ↓
Gimbal Program
     ↓
Gimbal Validation / Typing
     ↓
Target-Language Functor
     ↓
Target Project
     ↓
Builder
     ↓
Built Artifact
     ↓
optional Runner
```

This replaces the earlier direct architecture:

```text
Dream source
    ↓
LLM Composer
    ↓
target project
```

The direct Composer remains useful as an early implementation strategy, but Gimbal should eventually replace it as the semantic boundary.

## `dream now`

`dream now` remains intentionally different.

It can continue using the direct interpreter path:

```text
Dream Project
    ↓
Source Resolver
    ↓
LLM Interpreter
    ↓
Program Output
```

This preserves Dream's immediate, permissive execution mode.

Normal Dream and `dream now` therefore have different purposes:

```text
dream foo.foo
    → formalize through Gimbal
    → translate
    → build

dream now foo.foo
    → interpret immediately
```

## Semantic Locking

Once Gimbal is the semantic target, semantic locking becomes much simpler.

Instead of inventing a separate Dream IR or lock representation, Dream can persist the accepted Gimbal program.

```text
main.foo
   ↓
Dream interpretation
   ↓
Gimbal program
   ↓
semantic lock
```

That same locked meaning can then be translated repeatedly:

```text
Locked Gimbal Program
    ├── Rust
    ├── Go
    ├── C
    └── Python
```

This provides:

- reproducible semantics;
    
- cross-target consistency;
    
- reduced LLM usage;
    
- protection from model drift;
    
- inspectable accepted meaning;
    
- deterministic target translation.
    

## Strict Mode

Gimbal also gives strict mode a stronger eventual definition.

Rather than only telling the model to be conservative, strict Dream can require:

> Dream must resolve the pseudocode into a valid, fully typed Gimbal program without unresolved semantic assumptions.

Conceptually:

```text
Dream source
    ↓
semantic interpretation
    ↓
Can a valid Gimbal program be constructed?
    ├── yes → continue
    └── no  → DreamError
```

This gives strict mode a formal boundary instead of making it purely prompt-driven.

## Cross-Target Consistency

A major advantage of the Gimbal architecture is that target languages no longer receive independent interpretations.

Without Gimbal:

```text
Dream
 ├── LLM interpretation → Rust
 ├── LLM interpretation → Go
 └── LLM interpretation → Python
```

Those three outputs may subtly disagree.

With Gimbal:

```text
Dream
   ↓
one semantic interpretation
   ↓
Gimbal
 ├── Rust functor
 ├── Go functor
 └── Python functor
```

All targets begin from the same formal program.

This should make cross-target equivalence substantially easier to define, test, and eventually prove.

## Final Long-Term Model

```text
                       Dream
                         │
                         ▼
                Informal Pseudocode
                         │
                         ▼
               Semantic Interpretation
                         │
                         ▼
                       Gimbal
                         │
                  Formal Program
                         │
                Categorical Semantics
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       F_Rust          F_Go           F_C
          │              │              │
          ▼              ▼              ▼
    Rust Project     Go Project      C Project
          │              │              │
          ▼              ▼              ▼
       Builder         Builder        Builder
```

The long-term relationship between the two languages is therefore:

> **Dream is the informal semantic frontend. Gimbal is the formal categorical core.**

Dream lets the programmer describe the program naturally.

Gimbal makes that program exact.

Target-language functors carry that exact program into conventional software ecosystems.