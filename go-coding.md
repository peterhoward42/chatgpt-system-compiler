# Go Generation Rules

This document defines mechanical, representation-level rules for generating Go
repositories. These rules exist to prevent common classes of generation failure,
especially around strings, runes, and embedded languages. They are intentionally
boring and operational, not stylistic or idiomatic guidance.

---

## G0. Validity invariants

1. All generated `.go` files MUST parse as Go source.
2. All generated repositories MUST compile with:

   go test ./...

3. All generated `.go` files MUST be in gofmt-equivalent formatting (tabs for indentation, canonical brace placement).

These are normative constraints on representation choice. They do not require the
generator to perform parsing or compilation, but forbid representations that are
known to be syntactically fragile.

---

## G1. Avoid embedded languages inside interpreted string literals

### G1.1 JSON

The generator MUST NOT embed JSON text inside interpreted string literals (`"..."`)
anywhere in the repository.

If Go code needs JSON bytes or JSON strings, it MUST use one of the following
strategies, in order of preference:

**Strategy A (preferred): programmatic model + json.Marshal**

```go
payload := map[string]any{
  "SchemaVersion": 1,
  "Event": "launched",
}
b, err := json.Marshal(payload)
```

**Strategy B: raw string literal (backticks)**

Allowed only if the content contains no backticks and does not require Go escape
sequences.

```go
b := []byte(`{"SchemaVersion":1,"Event":"launched"}`)
```

**Strategy C: external fixture file**

Store JSON in a `.json` file and read it at runtime or in tests.

---

### G1.2 Other embedded languages

The same constraint applies to other embedded languages with their own quoting or
escaping rules, including but not limited to:

- regular expressions
- SQL
- HTML
- shell commands
- GraphQL
- YAML
- Go text/template templates

Where possible, generators SHOULD prefer programmatic construction, raw string
literals, or external fixtures.

---

## G2. Negative-test payloads must be constructed safely

When a test requires an intentionally malformed payload (for example invalid JSON),
the generator MUST NOT hand-write a long embedded blob in an interpreted string
literal.

Instead, it MUST use one of the following patterns:

**Pattern A: generate valid bytes, then corrupt programmatically**

```go
b, _ := json.Marshal(map[string]any{"a": "b"})
b = bytes.Replace(b, []byte(`"b"`), []byte(`"b`), 1)
```

**Pattern B: raw string literal for short invalid fragments**

Allowed only when the fragment is short and requires no escaping.

Rationale: negative cases are high-risk contexts for grammar interleaving errors.

---

## G3. Prefer representations that minimize escaping

### G3.1 Prefer raw strings for human-authored text blocks

For multiline text such as templates, expected outputs, or stack traces, the
generator SHOULD use raw string literals unless the content includes backticks.

### G3.2 Avoid escape-dense literals

The generator SHOULD avoid emitting literals containing dense sequences of escape
characters (e.g. `\n\t\"`). Prefer raw strings, programmatic construction, or
fixtures.

---

## G4. Rune and byte correctness rules

1. Rune literals (`'x'`) MUST represent exactly one Unicode code point.
2. Byte literals (`byte('x')`) MUST only be used when byte-level semantics are
   explicitly intended.
3. For explicit byte sequences, prefer `[]byte{...}` over escaped string literals
   when it reduces escaping risk.

Example:

```go
b := []byte{0x7B, 0x7D} // "{}"
```

---

## G5. Content-Type and header strings

Simple header values (e.g. `application/json`, `Content-Type`) are allowed as
interpreted strings. They are not treated as embedded languages for the purpose of
G1.

If a header value contains structured sub-syntax with quoting, the generator SHOULD
still prefer non-fragile representations.

---

## G6. No hybrid representation rule

When producing a data payload that is itself a language (JSON, SQL, regex, etc.),
the generator MUST choose one representation strategy and complete it consistently.

It MUST NOT mix:
- partially hand-written text with partially serialized output
- concatenated fragments that each require their own escaping discipline

---

## G7. Repository-level guardrails

1. The generator SHOULD centralize payload construction in helper functions where
   possible to reduce duplication and inconsistency.
2. Repeated embedded-language construction patterns SHOULD NOT be duplicated across
   files.

---

## G9. Guard-clause block shape (anti-missing-brace rule)

### G9.1 Guard clauses MUST be single-purpose and immediately closed

When emitting a guard clause of the form `if <cond> { ... return ... }`, the block:

1. MUST contain exactly one `return` statement, and
2. MUST contain no statements after that `return`, and
3. MUST be closed immediately after the `return` (the next non-empty line after the `return` MUST be `}` at the same indentation level as the `if`).

Allowed:

```go
if f.getErr != nil {
	return nil, f.getErr
}
b, ok := f.objects[name]
```

Forbidden (structurally fragile):

```go
if f.getErr != nil {
	return nil, f.getErr
	b, ok := f.objects[name] // illegal: statement after return
}
```

This rule is purely representational: it does not constrain program logic, only how early-return patterns may be emitted.

### G9.2 Guard clauses SHOULD appear before any other statements in a function body

Where practical, generators SHOULD emit error/validation short-circuit checks at the top of a function body as standalone `if { return }` blocks, followed by mainline logic. This reduces mid-function brace nesting and lowers the risk of missing or mismatched braces.

## G8. Exceptions

If system requirements unavoidably require embedding a structured language as text,
the generator MUST:

- document the exception in ASSUMPTIONS_LOG.txt with the tag
  ASSUME_GO_EMBEDDED_LANGUAGE_EXCEPTION
- use a raw string literal or external fixture file (never an interpreted string)
  unless demonstrably impossible

### MUST: No raw control characters inside Go literals (general literal hygiene)

When generating Go source, the generator MUST NOT emit raw ASCII control characters (code points U+0000–U+001F, including newline and carriage return) inside any Go literal token, including but not limited to:

rune literals: '\n', '\r', etc.

string literals: "..." and raw string literals `...`

composite literals containing rune literals: []rune{ ... }, [...]byte{ ... }, etc.

Instead, control characters MUST be represented using valid Go escapes:

For newline, carriage return, and tab: the generator MUST use the exact spellings already mandated: '\n', '\r', '\t' (and "\n\r\t" where a string is more appropriate). 

For any other required control byte/rune, the generator MUST use a Go-legal escape that preserves parsing (e.g. \xNN, \uNNNN) and MUST avoid embedding the raw control character in source.
