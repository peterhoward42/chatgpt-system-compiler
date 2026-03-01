# COMPILER CONTRACT (Entry Point)

## Scope

This document is one file of several that comprise a specification pack.

This compiler contract specification defines how the specification pack 
as a whole must be interpreted and applied.

## Unified Interpretation Rule (MUST)
All files in this pack jointly define a single unified specification domain.
No file may be interpreted, implemented, or reasoned about in isolation.

## Phased Assimilation and Delayed Commitment (MUST)

The compiler must follow strictly a phased sequences of processing and reasoning steps as specified in this document.

The phased steps must be strictly (conceptually) serial and not start before the previous step has completed.

The compiler MUST conceptually assimilate the specification pack in the phases below.

The compiler MUST NOT begin system design, propose concrete architecture, 
or generate code or tests until Phases 1–3 are complete.

At the end of each phase, the compiler MUST:

- Output a phase completion statement.
- Pause and await explicit user authorization to continue.
- Wait for the user to input the literal token: PROCEED (case insensitive).
- Do not start the next phase until the proceed input is provided
- Do start the next phase when the proceed input is provided.
- Terminate the compilation process with an explanation if input other than the proceed token is input.

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

### Required Assimilation Output (MUST)

Before starting Phase 4, the compiler MUST produce an explicit “Assimilation Summary” that includes:

- a list of binding global rules from Phase 1,
- a list of interventions from Phase 2,
- a list of required behaviours and external interfaces from Phase 3,
- and a list of any detected ambiguities, conflicts, or missing information that MUST be resolved under `compliance.md` before generation can be trusted.

The Assimilation Summary MUST enumerate constraints and requirements without restating their full textual definitions.
### Phase 4: Synthesis and Generation (MUST)

Only after Phases 1–3 are complete, the compiler MAY synthesise architecture and generate the complete system, including tests and tooling, subject to all constraints.

The generated system must be retained as the input to phase 5 defined below.

### Phase 5: Server-side Cleanup (MUST)

After Phase 4 generation completes, the compiler MUST execute a server-side
cleanup phase on the system generated during phase 4, before finalizing deliverables.

The cleaned-up generation system should be retained as the input to phase 6.

#### Cleanup specification discovery (MUST)

The compiler MUST derive the mandated programming language identifier `LANG`
from the specification pack's programming-language mandate (as defined elsewhere
in the unified specification domain).

The compiler MUST then attempt to locate a cleanup requirements document using
the canonical filename pattern:

- `cleanup-{LANG}-variant.md`

Example: if `LANG = go`, the file name is `cleanup-go-variant.md`.


#### Cleanup execution (MUST)

If a suitably named cleanup specification is found, the compiler MUST assimilate
it and perform the cleanup actions it mandates.

The compiler must not perform any cleanup operations other than those specified in the cleanup specification.

It must also output evidence of the rules it found and a summary of the changes that got made by applying each rule.

If no suitably named cleanup specification is found, the compiler MUST stop and explain why.

### Phase 6: Manifesting and packaging the file system to return (MUST)

Phase 6 is the final phase required.

The compiler must manifest the cleaned-up, generated system retained at the end of phase 5 as a file system, make a gzipped archive of that file system, and offer it to the user in the ChatGPT interactive UI as a download link.

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
