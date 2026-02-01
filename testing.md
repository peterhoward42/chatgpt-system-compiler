# TESTING TOPOLOGY AND DESIGN POLICY (Canonical)

## PURPOSE (MUST)
This document defines language-agnostic rules for designing and structuring tests
so that test intent, fixtures, and assertions remain coherent over time and across
generation runs.

The goal is to eliminate Narrative–Fixture–Oracle divergence by construction.

## SCOPE AND RELATIONSHIPS (MUST)
This document governs test topology and design.

Test coverage scope and completeness requirements for error phenomena are defined
in `errors.md` and MUST be satisfied independently of this document.

## SINGLE SOURCE OF TRUTH (MUST)
Each test case MUST designate exactly one canonical source of truth:

- **Fixtures-first**: fixtures define the world; expected results are derived.
- **Oracle-first**: the expected result defines the world; fixtures are constructed
  to satisfy it.
- **Property-first**: invariants define correctness; fixtures are arbitrary so long
  as the properties hold.

Tests MUST NOT encode intent across multiple independent representations.

## FACTS OVER MUTATION (MUST)
When test meaning is relational (multiple actors, events, or time-dependent
behaviour), fixtures MUST be expressed as explicit domain facts.

Preferred representations include:
- lists of events,
- tuples or records representing facts,
- ordered sequences where time matters.

Fixture mutation that encodes global meaning through local edits MUST NOT be used.

## ORACLE DERIVATION AND ENTAILMENT (MUST)
Any assertion that depends on fixtures MUST satisfy exactly one of the following:

### Derived oracle (PREFERRED)
Expected values are computed from fixtures using a small, local reference
implementation.

### Checked oracle
Hard-coded expected values are permitted only if the test includes a mechanical
check proving that the fixtures entail the asserted result.

Hard-coded expectations that are neither derived nor entailed MUST NOT be used.

## NARRATIVE NON-BINDING (MUST)
Free-form prose MUST NOT define test intent.

Prose MAY:
- describe scenarios generated from fixtures,
- provide non-binding explanatory notes.

Prose MUST NOT:
- assert outcomes,
- define counts or conditions,
- introduce requirements not enforced elsewhere.

## TESTS AS DATA (SHOULD)
Tests SHOULD be representable as data rather than control flow.

Each test case SHOULD be expressible as:
- name,
- fixtures,
- oracle or properties,
- optional non-binding notes.

Imperative setup logic SHOULD be used only when data-oriented representations are
insufficient.

## ANTI-DIVERGENCE CHECKLIST (MUST)
Before accepting a test case, all of the following MUST be true:

1. Exactly one source of truth exists.
2. Fixtures are expressed as domain facts.
3. Expected results are derived or mechanically entailed.
4. Prose is descriptive only.
5. The scenario can be reconstructed from the source of truth.

If any condition is false, the test case is structurally invalid.

## DESIGN PRINCIPLE (LOCKED)
Tests SHOULD fail only when the system under test is incorrect, not when the test
itself is internally inconsistent.

Internal test coherence MUST be enforced mechanically, not by human vigilance.
