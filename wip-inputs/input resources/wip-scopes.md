Proposed document scopes (7 total)
1. System Behaviour Specification

(What the system does at runtime)

Scope
This document defines the deployed telemetry system as a black box:

what inputs it accepts,

what outputs it produces,

what invariants it enforces,

and what guarantees it makes.

Includes

System overview

HTTP interface

Telemetry event model

Payload validation rules

Persistence strategy and object naming

Analysis interface, schema, and semantics

Security and abuse model

Operational assumptions (explicitly marked as assumptions)

Non-goals related to runtime behaviour

Excludes

How code is generated

How tests are written

Repository layout

How humans collaborate with ChatGPT

Why it’s coherent
A reader could use only this document to understand what the deployed service does and to judge whether an implementation is correct, regardless of how it was produced.

2. Generation & Collaboration Contract

(How ChatGPT is expected to behave as a generator)

Scope
This document defines ChatGPT’s obligations and freedoms when acting as a generator.

Includes

“ChatGPT must behave as a compiler”

Full regeneration requirement

Iteration mechanics

Ambiguity resolution policy

Assume-don’t-ask rule

Trade-off preference rules

Partial output rules

Deviation register rules

Commenting rules (as generator obligations)

Excludes

Any specific system behaviour

Test mechanics

Tooling details

Why it’s coherent
This is the contract between human and generator. It answers: “What kind of collaborator is ChatGPT supposed to be?”

3. Correctness, Compliance & Evidence Rules

(How correctness is demonstrated and failures are handled)

Scope
This document defines what it means for a generated system to be compliant with the specification and how violations are surfaced.

Includes

MUST / SHOULD compliance semantics

Conflict and infeasibility handling

Deviation register definition and contents

Testing claims safety rule

Rules about what may and may not be claimed

Excludes

Actual test structure

System behaviour details

Generator heuristics

Why it’s coherent
This scope is about epistemology: how we know whether the system meets the spec, and how we talk honestly when it doesn’t.

4. Testing & Testability Contract

(How tests must be structured and what they must prove)

Scope
This document defines the shape of acceptable tests and the constraints imposed on the implementation to make those tests possible.

Includes

Public-interface-first testing

HTTP-driven testing rules

Reachability of business-rule branches

Fakes-not-reimplementation rule

Optional internal unit tests

Injectability requirements (time, randomness, ports)

Excludes

How the generator chooses architectures

Runtime system semantics

Tooling and Makefiles

Why it’s coherent
This is the bridge between spec and code, but focused purely on verification, not generation.

5. Tooling & Repository Artefacts

(What must exist alongside the code)

Scope
This document defines the non-code artefacts that accompany a generated system.

Includes

Repository artefacts rules

Inclusion of the spec verbatim

README constraints (pointer-only)

Makefile expectations

Assumptions log

Any future CI / deployment artefacts (if added later)

Excludes

Runtime behaviour

Test semantics

Generator behaviour

Why it’s coherent
This is about shape and hygiene of the repository, not the system or the generator.

6. Platform & Deployment Constraints

(Where and how the system runs)

Scope
This document defines hard constraints imposed by the target platform.

Includes

Google Cloud Functions model

Single function, single endpoint

Entry point naming

Go toolchain requirements

Configuration policy (compile-time constants)

Logging policy

Excludes

Telemetry semantics

Testing rules

Collaboration mechanics

Why it’s coherent
These constraints are externally imposed and largely orthogonal to system intent. Keeping them isolated prevents accidental scope creep.

7. Specification Canonicalisation & Evolution Policy

(How specs are transformed and maintained over time)

Scope
This document defines how messy, exploratory specs become canonical ones.

Includes

Non-lossiness invariant

Orthogonalisation requirement

Semantic tagging rules

Assume-don’t-ask during canonicalisation

Structural and formatting rules

Inline notes policy

Ancillary requirements sink

Output requirements

Excludes

Any DrawExact-specific system behaviour

Generator runtime obligations

Why it’s coherent
This is a meta-spec governing the evolution of other specs, not the system itself.