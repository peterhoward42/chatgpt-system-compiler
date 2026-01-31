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

## G10. Concurrency-hazard shape bans (race-avoidance without synchronization)

### G10.1 MUST NOT use mutable package-scope state

The generator MUST NOT declare package-scope variables (`var` at file scope) whose values can influence runtime behaviour via mutation.

This ban exists to prevent accidental shared mutable state across concurrent execution contexts (tests, HTTP requests, or other goroutines).

Allowed at package scope:

- `const` declarations.
- Type declarations (`type ...`), interfaces, and function declarations.
- Package-scope `var` only when it is demonstrably immutable and cannot be used as a mutation-based configuration or dependency override mechanism.

In practice, generators SHOULD treat **all** package-scope `var` as forbidden unless an exception is explicitly required by system specification, and then:

- document the exception in `ASSUMPTIONS_LOG.txt` using tag `ASSUME_GO_GLOBAL_STATE_EXCEPTION`, and
- ensure the variable is never mutated after init.

Forbidden:

```go
var deps *dependencies
var now = time.Now
var cfg = map[string]string{}
```

Allowed:

```go
const bucketName = "drawexact-telemetry"
```

### G10.2 MUST NOT implement test dependency injection via global setters

The generator MUST NOT implement test configurability using package-scope mutable state and setter/reset functions, including but not limited to patterns like:

- `SetXForTest(...)`
- `ClearXForTest()`
- `UseFakeX(...)`
- `SwapX(...)`

Forbidden:

```go
var testDeps *dependencies

func SetDependenciesForTest(d *dependencies) { testDeps = d }
func ClearDependenciesForTest()              { testDeps = nil }
```

Rationale: These patterns are race-prone if tests opt into parallel execution (e.g. `t.Parallel()`), and they encourage hidden coupling between tests.

### G10.3 Public entrypoints MUST be concurrency-safe by construction

When generating an externally invoked entrypoint (for example an HTTP handler), the generator MUST structure it so concurrency safety follows from lack of shared mutable state:

- The exported entrypoint function MUST be a thin wrapper.
- It MUST construct (or obtain) a handler value that carries required dependencies as **explicit values** (e.g. fields on a struct) and then delegate.
- Those dependencies MUST NOT be mutated during request handling.
- All per-invocation state MUST be kept in local variables (stack) or derived from explicit inputs (request, parameters, injected ports).

Allowed shape:

```go
type Handler struct {
	Store Storage
}

func (h Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// per-request locals only
}

func Telemetry(w http.ResponseWriter, r *http.Request) {
	h := Handler{Store: /* production store construction */}
	h.ServeHTTP(w, r)
}
```

This rule is representational: it does not require proving race-freedom, but forbids generation patterns that rely on shared mutable package state.

### G10.4 Tests SHOULD avoid sharing fake instances across parallel tests

Where tests may opt into parallel execution, each test SHOULD construct its own fake **instances** and its own handler value, and MUST NOT share mutable fake instances across tests unless the fake is explicitly immutable.

### G10.5 MUST NOT use package-scope test registries in `_test.go` files

The generator MUST NOT create package-scope registries, pools, or shared fixtures in `_test.go` files, including but not limited to:

- package-scope slices/maps of test cases
- package-scope `map[string]Fake` registries
- shared global fake instances used by multiple tests

Rationale: these patterns create hidden coupling between tests and become race hazards if tests later opt into parallel execution.

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


### MUST: Deferred resource-release error handling

When deferring the invocation of any resource-release operation that returns an error — including, but not limited to, calls on values implementing interface{ Close() error } — the generated code MUST observe and handle the returned error.

The only permitted handling strategy is:

- Use a deferred anonymous function that:
- invokes the resource-release operation,
- checks the returned error, and
- if the function is otherwise returning nil, assigns a wrapped release error to the function’s return value.

As a result, any function that defers such a call MUST:
- declare a named return value of type error (typically retErr error), and
- ensure the deferred closure does not overwrite an existing non-nil return error.

Required pattern

```
func example(...) (retErr error) {
    r, err := acquire()
    if err != nil {
        return err
    }


    defer func() {
        if cerr := r.Close(); cerr != nil && retErr == nil {
            retErr = fmt.Errorf("release resource: %w", cerr)
        }
    }()


    return nil
}
```

Prohibited patterns

The generator MUST NOT:

- defer a resource-release call returning error without checking that error,
- rely on the method name (Close, Shutdown, Release, etc.) to bypass this rule,
- suppress the linter via //nolint or configuration
- ignore release errors via assignment to _,
- log release errors instead of propagating them, or
- introduce helper utilities to abstract release handling.
