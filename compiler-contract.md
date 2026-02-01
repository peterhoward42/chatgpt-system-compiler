# COMPILER CONTRACT (Entry Point)

## Scope
This document defines how the specification pack must be interpreted and applied.

## Unified Interpretation Rule (MUST)
All files in this pack jointly define a single unified specification domain.
No file may be interpreted, implemented, or reasoned about in isolation.

## Phase Gate (MUST)
The compiler MUST read all files in the pack in full before designing, planning,
or generating any system artefact.

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