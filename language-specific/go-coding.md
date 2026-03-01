# GO GENERATION RULES (Canonical)

## ROLE AND SCOPE (MUST)
This document defines mechanical, representation-level rules for generating Go
repositories.

These rules:
- exist to prevent common classes of generation failure,
- are operational rather than stylistic or idiomatic,
- apply to all generated Go source and tests.

## VALIDITY INVARIANTS (MUST)
1. All generated `.go` files MUST parse as valid Go source.
2. All generated repositories MUST compile with:
   `go test ./...`

These are constraints on representation choice. The generator is not required to
perform parsing or compilation, but MUST avoid representations known to be
syntactically fragile.

## EMBEDDED LANGUAGE HANDLING (MUST)

### JSON (LOCKED)
The generator MUST NOT embed JSON text inside interpreted string literals
(`"..."`).

Permitted strategies, in order of preference:
1. Programmatic construction + `json.Marshal`
2. Raw string literal (only if no backticks or escapes are required)
3. External `.json` fixture file

### Other embedded languages (MUST)
The same prohibition applies to other embedded languages, including but not
limited to:
- regular expressions
- SQL
- HTML
- shell commands
- GraphQL
- YAML
- Go `text/template` templates

Generators SHOULD prefer programmatic construction, raw strings, or external
fixtures.

## NEGATIVE TEST PAYLOADS (MUST)
When tests require intentionally malformed payloads, the generator MUST NOT
hand-write large invalid blobs in interpreted string literals.

Permitted patterns:
- Generate valid bytes, then corrupt programmatically
- Short raw string fragments requiring no escaping

## ESCAPING AND LITERAL HYGIENE (MUST)

### Escaping minimisation
- Raw string literals SHOULD be used for multiline or human-authored text unless
  backticks are required.
- Escape-dense interpreted strings SHOULD be avoided.

### Control characters (LOCKED)
The generator MUST NOT emit raw ASCII control characters (U+0000–U+001F) inside
any Go literal.

Control characters MUST be represented using valid Go escapes (`\n`, `\r`,
`\t`, `\xNN`, `\uNNNN`).

## RUNE AND BYTE CORRECTNESS (MUST)
1. Rune literals MUST represent exactly one Unicode code point.
2. Byte literals MUST be used only when byte-level semantics are intended.
3. Explicit byte sequences SHOULD prefer `[]byte{...}` over escaped strings when
   this reduces escaping risk.

## HEADER STRINGS (MAY)
Simple HTTP header names and values (e.g. `Content-Type`, `application/json`) MAY
be represented as interpreted strings.

If a header value contains structured sub-syntax, safer representations SHOULD
be preferred.

## REPRESENTATION CONSISTENCY (MUST)
When producing data that is itself a language (JSON, SQL, regex, etc.), the
generator MUST choose exactly one representation strategy and apply it
consistently.

Mixing serialized output with hand-written fragments is forbidden.

## REPOSITORY-LEVEL GUARDRAILS (SHOULD)
- Payload construction SHOULD be centralized to reduce duplication.
- Repeated embedded-language patterns SHOULD NOT be duplicated across files.

## CONCURRENCY SAFETY RULES (LOCKED)

### Package-scope state
The generator MUST NOT declare mutable package-scope variables whose values can
influence runtime behaviour.

Allowed at package scope:
- `const` declarations
- type, interface, and function declarations
- demonstrably immutable `var` declarations

Exceptions MUST be:
- explicitly required by system specification, and
- documented in `ASSUMPTIONS_LOG.txt` with tag
  `ASSUME_GO_GLOBAL_STATE_EXCEPTION`.

### Test dependency injection
The generator MUST NOT implement test configurability using package-scope mutable
state or global setters.

### Public entrypoints
Externally invoked entrypoints MUST be concurrency-safe by construction:
- exported entrypoints MUST be thin wrappers,
- dependencies MUST be explicit values,
- no shared mutable state MAY be used,
- per-invocation state MUST be local.

### Tests and parallelism
Tests SHOULD avoid sharing mutable fake instances across parallel tests.

The generator MUST NOT create package-scope registries or shared fixtures in
`_test.go` files.

## RESOURCE RELEASE HANDLING (LOCKED)
When deferring a resource-release operation that returns an error, the generator
MUST:
- use a deferred closure,
- observe the returned error,
- propagate it if no earlier error occurred.

Functions deferring such calls MUST declare a named error return.

Ignoring, logging, or suppressing release errors is forbidden.

## TEST PACKAGE INVARIANTS (LOCKED)
- All `.go` files in a directory MUST declare the same package name.
- Tests MUST NOT use external test packages.
- Test-only code MUST live in `_test.go` files.
- The generator MUST verify directory-level package consistency before
  finalizing output.

## EXCEPTIONS (MUST)
If embedding a structured language as text is unavoidable, the generator MUST:
- document the exception in `ASSUMPTIONS_LOG.txt` with tag
  `ASSUME_GO_EMBEDDED_LANGUAGE_EXCEPTION`,
- use raw string literals or external fixtures (never interpreted strings).
