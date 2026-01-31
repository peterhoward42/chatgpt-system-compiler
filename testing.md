# Testing Topology and Design Policy

This document defines **language-agnostic rules for designing and structuring tests**
so that test intent, fixtures, and assertions remain coherent over time and across
generation runs.

The goal is to eliminate *Narrative–Fixture–Oracle divergence* by construction.

---
## 0. Relationship to `errors.md`

This document governs **test topology and design**.
Test **coverage scope and completeness** requirements for error phenomena are defined in `errors.md` and MUST be satisfied.

---

## 1. Single Source of Truth Rule

Each test case **must designate exactly one canonical source of truth**:

- **Fixtures-first**  
  Fixtures define the world; expected results are *derived* from them.

- **Oracle-first**  
  The expected result defines the world; fixtures are constructed to satisfy it.

- **Property-first**  
  Invariants define correctness; fixtures are arbitrary as long as properties hold.

**Forbidden:**  
Tests where prose describes one scenario, fixtures instantiate another, and assertions
encode a third.

> **Rule:** A test case must have exactly one canonical representation of intent.  
> All other representations must be derived from it or mechanically checked against it.

---

## 2. Facts Over Mutation Rule

When test meaning is relational (multiple actors, events, or time-dependent behavior),
fixtures must be expressed as **explicit domain facts**, not as incremental mutation of
a base object.

**Prefer:**
- Lists of events
- Tuples or records representing domain facts
- Ordered sequences when time matters

**Avoid:**
- “Copy valid object and overwrite one field”
- Fixture mutation that encodes global meaning through local edits

> **Rule:** If a scenario requires reasoning across multiple fixtures, fixtures must be
declared as an explicit list of domain facts.

---

## 3. Derived or Checked Oracle Rule

Any assertion that depends on fixtures **must satisfy one of the following**:

### 3A. Derived Oracle (Preferred)
Expected values are computed from fixtures using a small, local reference implementation.

### 3B. Checked Oracle
Hard-coded expected values are allowed **only if** the test includes a check proving
that the fixtures entail the asserted result.

> **Rule:** Hard-coded expectations derived from fixtures must be either programmatically
derived or mechanically entailed by the fixtures.

---

## 4. Narrative Is Non-Binding Rule

Free-form prose creates an additional “truth surface” and must not introduce
requirements not enforced elsewhere.

Allowed:
- Descriptive comments generated from fixtures
- Notes that do not assert quantitative or logical outcomes

Forbidden:
- Prose asserting counts, conditions, or outcomes not mechanically enforced

> **Rule:** Prose must never be the only place where test intent is encoded.

---

## 5. Tests as Data Rule

Prefer table-driven or data-oriented test structures:

Each test case should be representable as:
- `name`
- `fixtures`
- `oracle` **or** `properties`
- optional `notes` (non-binding)

Imperative setup logic should be considered a last resort.

> **Rule:** Test scenarios should be representable as data structures, not control flow.

---

## 6. Anti-Divergence Checklist

Before accepting a test case, verify:

1. Is there exactly one source of truth?
2. Are fixtures expressed as domain facts?
3. Are expected values derived or entailed?
4. Do comments merely describe, not define?
5. Can the scenario be reconstructed from one place?

If any answer is **no**, the test is structurally unstable.

---

## Design Principle

**Tests should fail when the system is wrong — not when the test has confused itself.**

This policy exists to ensure that internal test coherence is enforced mechanically,
not by human vigilance.
