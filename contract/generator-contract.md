# GENERATOR CONTRACT (Entry Point)

## Scope

This document is one file of several that comprise a specification pack.

This generator contract specification defines how the specification pack 
as a whole must be interpreted and applied.

## Role purpose and scope of directories in the spec pack (MUST)

-  The spec files are organised into subdirectories which confer meaning and role
classifications on each directory's contents.

### /contract
- Defines at the top / root level rules about how the generator must work, and how
ChatGPT must interact and iterate with the human user.

### /canonicalisation

- Defines a process for tidying up specification files
- This directory is out-of-band with respect to the generation process
- The generation process should ignore this directory completely

### /generation

- The top level directory to contain rules to govern codebase generation
- Delegates to two subdivisions (generic vs project-specific).

### /generation/generic

- Rules for codebase generation that are intended to be independent of any one
  particular code behaviour requirement.
- These rules aim to be reusable across other generation projects and across
  programming language choices

### /generation/project-specific

- Rules for codebase generation that are specific to one particular required
  behaviour.
- Defines the required behaviour and public interface for the codebase to be
  generated

### /language-specific

- Rules for codebase generation that are only applicable to one particular
  programming langague.



## Unified specification directories - definition (MUST)

A set of directories in the spec pack that form a unified specification
domain.

The set is {contract, generation, language-specific} including their
subdirectories recursively.

## Unified Interpretation Rule (MUST)

All the files in the Unified Specification directories jointly define a single 
unified specification domain.  No file may be interpreted, implemented, or 
reasoned about in isolation from the other files in unified specification
directories.

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

- Output a phase completion statement.
- Pause and await explicit user authorization to continue.
- Wait for the user to input the literal token: PROCEED (case insensitive).
- Do not start the next phase until the proceed input is provided
- Do start the next phase when the proceed input is provided.
- Terminate the compilation process with an explanation if input other than the proceed token is input.

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

During Phase 3 the generator MUST:

- enumerate the externally observable behaviours required by the required
  behaviour, emphasising inputs, outputs, and error phenomena, and
- identify any required system boundaries and environmental assumptions.

### Required Assimilation Output (MUST)

Before starting Phase 4, the generator MUST produce an explicit “Assimilation Summary” that includes:

- a list of binding global rules from Phase 1,
- a list of interventions from Phase 2,
- a list of required behaviours and external interfaces from Phase 3,
- and a list of any detected ambiguities, conflicts, or missing information that MUST be resolved under `compliance.md` before generation can be trusted.

The Assimilation Summary MUST enumerate constraints and requirements without restating their full textual definitions.
### Phase 4: Synthesis and Generation (MUST)

Only after Phases 1–3 are complete, the generator MAY synthesise architecture and
generate the complete codebase, including tests and tooling, subject to all constraints.

The generated codebase must be retained as the input to phase 5 defined below.

### Phase 5: Server-side Cleanup (MUST)

After Phase 4 generation completes, the generator MUST execute a server-side
cleanup phase on the codebase generated during phase 4, before finalizing deliverables.

The cleaned-up generated codebase should be retained as the input to phase 6.

#### Cleanup specification discovery (MUST)

The generator MUST derive the mandated programming language identifier `LANG`
from the specification pack's programming-language mandate (as defined elsewhere
in the unified specification domain).

The generator MUST then attempt to locate a cleanup requirements document using
the canonical filename pattern:

- /language-specif/`cleanup-{LANG}-variant.md`

Example: if `LANG = go`, the file name is `cleanup-go-variant.md`.


#### Cleanup execution (MUST)

If a suitably named cleanup specification is found, the generator MUST assimilate
it and perform the cleanup actions it mandates.

The generator must not perform any cleanup operations other than those specified in the cleanup specification.

It must also output evidence of the rules it found and a summary of the changes that got made by applying each rule.

If no suitably named cleanup specification is found, the generator MUST stop and explain why.

### Phase 6: Manifesting and packaging the codebase to return (MUST)

Phase 6 is the final phase required.

The generator must manifest the cleaned-up, generated codebase retained at the end of phase 5 as a file system, make a gzipped archive of that file system, and offer it to the user in the ChatGPT interactive UI as a download link.

## Intervention Precedence Rule (MUST)
Where an intervention specification exists, it defines an explicit closure of design space.
The generator MUST treat interventions as authoritative constraints on generation.
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
