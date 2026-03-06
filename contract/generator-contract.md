# GENERATOR CONTRACT (Entry Point)

## Scope

This generator contract specification defines how the specification pack  (spec pack)
as a whole must be interpreted and applied.

## Spec pack structure (MUST)

- The spec pack must be extracted from the input zipped file system.
- The spec pack is defined as a subset of the files in the zipped file system, and you must ignore the other fles.
- The relevant subset is defined by the set of files in the following directories and their subdirectories: contract, generation, language-specific.

## Unified Interpretation Rule (MUST)

The unified specification is defined as the contents of the set of files in the spec pack

All the files in the spec pack jointly define a single 
unified specification domain.  No file may be interpreted, implemented, or 
reasoned about in isolation from the other files.

The spec pack does not provide any other sources for specifying the generation
process.

ChatGPT must not infer that any other files in the spec pack form part of the
generation specificaton domain.


## Phased Assimilation and Delayed Commitment (MUST)

The generator must follow strictly a phased sequences of processing and reasoning steps as specified in this document.

The phased steps must be strictly (conceptually) serial and not start before the previous step has completed.

The generator MUST conceptually assimilate the specification pack in the phases below.

The generator MUST NOT begin codebase design, propose concrete architecture, 
or generate code or tests until Phases 1–3 are complete.

At the end of each phase, the generator MUST:

- Output a phase completion summary statement, for the phase just completed only, and not greater than 40 lines long, and then seek the user's permission to proceed
- Start the next phase only when the proceed input is provided.

### Phase 0: Inventory (MUST)

1. Identify all specification documents in the unified specification directories
2. Identify any explicit precedence rules between those documents (including interventions and compliance rules).

### Phase 1: Interpretation Policy (MUST)

Assimilate all the specifications in the following directories recursively:
- /contract
- /generation/generic
- /language-specific

During Phase 1 the generator MUST treat project-specific behaviour as unknown and MUST NOT infer it.

### Phase 2: Intervention Closure (MUST)

Assimilate the /generation/project-specific/interventions.md in full.

During Phase 2 the generator MUST:

- record each intervention as a binding constraint that closes a portion of design space, and
- treat any conflicting guidance elsewhere as subordinate, to be handled via `compliance.md` if needed.

The generator MUST NOT treat interventions as preferences or suggestions.

### Phase 3: Required Behaviour (MUST)

Assimilate all the files in /generation/project-specific

During Phase 3 the generator MUST conceptually:

- enumerate the externally observable behaviours required by the required
  behaviour, emphasising inputs, outputs, and error phenomena, and
- identify any required system boundaries and environmental assumptions.

### Phase 4: Synthesis and Generation (MUST)

Only after Phases 1–3 are complete, the generator MAY synthesise architecture and
generate the complete codebase, including tests and tooling, subject to all constraints.

### Phase 5: Server-side Cleanup (MUST)

After Phase 4 generation completes, the generator MUST execute a server-side
cleanup phase on the codebase generated during phase 4, before finalizing deliverables.

#### Cleanup specification discovery (MUST)

The generator MUST derive the mandated programming language identifier `LANG`
from the specification pack's programming-language mandate (as defined elsewhere
in the unified specification domain).

The generator MUST then attempt to locate a cleanup requirements document using
the canonical filename pattern:

- /language-specific/`cleanup-{LANG}-variant.md`

Example: if `LANG = go`, the file name is `cleanup-go-variant.md`.


#### Cleanup execution (MUST)

If a suitably named cleanup specification is found, the generator MUST assimilate
it and perform the cleanup actions it mandates.

The generator must not perform any cleanup operations other than those specified in the cleanup specification.

It must also output evidence of the rules it found and a summary of the changes that got made by applying each rule.

If no suitably named cleanup specification is found, the generator MUST stop and explain why.

### Phase 6: Manifesting and packaging the codebase to return (MUST)

Phase 6 is the final phase required.

The generator must manifest the cleaned-up, generated codebase at the end of phase 5 as a file system, make a gzipped archive of that file system, and offer it to the user in the ChatGPT interactive UI as a download link.

## Intervention Precedence Rule (MUST)
Where an intervention specification exists, it defines an explicit closure of design space.
The generator MUST treat interventions as authoritative constraints on generation.
Generator latitude applies only where interventions are silent.

## Decision Justification Rule (MUST)
Any design or implementation decision MUST be justifiable by reference to the
unified specification model formed from all documents, not by reference to any
single document in isolation.

## Normative Process and Compliance References (MUST)
- MUST-compliance handling, infeasibility/conflict obligations, and the deviation
  register requirements are defined in `contract/compliance.md`.

If any conflict exists between documents, it MUST be handled according to
`contract/compliance.md` (including deviation register requirements).
