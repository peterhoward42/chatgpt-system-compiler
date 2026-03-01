# Collaborating with ChatGPT as a Codebase (repo) Generator

## Project Status

Research milestone.\
Pre-alpha.\
Runnable.

This repository captures a snapshot in an ongoing research experiment
exploring whether an LLM can be treated as a specification-driven codebase
generator.

It is both:

-   A documented research milestone
-   A runnable pre-alpha codebase generation system

------------------------------------------------------------------------

## Research Objective

The central question:

Can a large language model generate a complete, high-quality codebase
from a specification alone: in a single shot, with strong
test coverage, and sound engineering practices?

The core inversion is simple:

You iterate the specification, not the code.\
Each iteration regenerates the entire repository.

------------------------------------------------------------------------

## Core Method

The generator is driven by a structured "spec pack" that includes:

-   Desired generated codebase behaviour contract
-   Programming language rules
-   Architecture and deployment constraints
-   Quality and testing expectations

The LLM is treated as a generation engine that consumes this
specification and produces a fully formed repository.

Regeneration replaces modification.
Refinement happens at the level of abstraction.

------------------------------------------------------------------------

## What This Repository Contains

-   A reusable specification framework
-   A built-in example required-behaviour specification
-   Lessons learned from iterative refinement
-   Strategies for mitigating LLM failure modes
-   A workflow for full regeneration

------------------------------------------------------------------------

## Results at This Milestone

At this stage, the experiment has demonstrated:

-   High-quality generated code under disciplined specification control
-   Strong test precision when constraints are carefully expressed
-   A substantial portion of the spec pack that appears reusable across
    projects

However, two consecutive generations from the same specification can
produce significantly different codebases. Output quality is
probabilistic rather than deterministic.

------------------------------------------------------------------------

## Known Limitations

### Non-Determinism

Generations are not reproducible at the byte level. Adequacy must be
evaluated probabilistically.

### Context Window Limits

The experiment encountered hidden contextual memory exhaustion. When
this occurred, the model silently deprioritized or forgot rules, causing
unpredictable regression in output quality.

Mitigation strategies are discussed in `lessons-learned.md`, but the
problem is not fully solved.

### Pre-Alpha Stability

This generator is experimental. Interfaces, structure, and workflow may
change.


## Try It

1.  Read `quick-start.md`
2.  Examine `generation/project-specific/required-behaviour.md`
3.  Adjust or replace the required behaviour specification
4.  Trigger a full regeneration workflow
5.  Inspect and evaluate the generated repository

------------------------------------------------------------------------

## Open Research Directions

-   Determinism strategies under probabilistic generation
-   Context partitioning and memory discipline
-   Spec abstraction levels that generalize across project classes
-   Tooling around regeneration validation

------------------------------------------------------------------------

This repository is intended as a foundation for further experimentation.
It is not a finished framework, but a structured milestone in exploring
LLM-driven codebase generation.
