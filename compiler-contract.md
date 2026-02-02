# COMPILER CONTRACT (Entry Point)

## Scope

This document is one file of several that comprise a specification pack.

This compiler contract specification defines how the specification pack 
as a whole must be interpreted and applied.

## Unified Interpretation Rule (MUST)
All files in this pack jointly define a single unified specification domain.
No file may be interpreted, implemented, or reasoned about in isolation.

## Phased Assimilation and Delayed Commitment (MUST)

The compiler MUST conceptually assimilate the specification pack in the phases below.

The compiler MUST NOT begin system design, propose concrete architecture, 
or generate code or tests until Phases 1–3 are complete.

### Phase 0: Inventory (MUST)

1. Identify all specification documents in the pack.
2. Identify any explicit precedence rules between documents (including interventions and compliance rules).

### Phase 1: Interpretation Policy (MUST)

Assimilate all reusable policy documents that define:

- normative language and modal meanings (MUST / SHOULD / MAY),
- collaboration and compliance obligations,
- validation and testing obligations,
- and any global invariants that apply across all projects.

During Phase 1 the compiler MUST treat project-specific behaviour as unknown and MUST NOT infer it.

### Phase 2: Intervention Closure (MUST)

Assimilate the Intervention Specification in full.

During Phase 2 the compiler MUST:

- record each intervention as a binding constraint that closes a portion of design space, and
- treat any conflicting guidance elsewhere as subordinate, to be handled via `compliance.md` if needed.

The compiler MUST NOT treat interventions as preferences or suggestions.

### Phase 3: Required System Behaviour (MUST)

Assimilate all project-specific specifications that define required runtime behaviour and external interfaces.

During Phase 3 the compiler MUST:

- enumerate the externally observable behaviours required by the system, emphasising inputs, outputs, and error phenomena, and
- identify any required system boundaries and environmental assumptions.

### Phase 4: Synthesis and Generation (MUST)

Only after Phases 1–3 are complete, the compiler MAY synthesise architecture and generate the complete system, including tests and tooling, subject to all constraints.

### Required Assimilation Output (MUST)

Before starting Phase 4, the compiler MUST produce an explicit “Assimilation Summary” that includes:

- a list of binding global rules from Phase 1,
- a list of interventions from Phase 2,
- a list of required behaviours and external interfaces from Phase 3,
- and a list of any detected ambiguities, conflicts, or missing information that MUST be resolved under `compliance.md` before generation can be trusted.

The Assimilation Summary MUST enumerate constraints and requirements without restating their full textual definitions.


## Intervention Precedence Rule (MUST)
Where an intervention specification exists, it defines an explicit closure of design space.
The compiler MUST treat interventions as authoritative constraints on generation.
Generator latitude applies only where interventions are silent.

## Decision Justification Rule (MUST)
Any design or implementation decision MUST be justifiable by reference to the
unified specification model formed from all documents, not by reference to any
single document in isolation.

## Normative Process and Compliance References (MUST)
- Generation behaviour and tooling/testing constraints are defined in `generation.md`.
- MUST-compliance handling, infeasibility/conflict obligations, and the deviation
  register requirements are defined in `compliance.md`.

If any conflict exists between documents, it MUST be handled according to
`compliance.md` (including deviation register requirements).
