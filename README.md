# README

## A Short Story: Collaborating with ChatGPT as a System Compiler

This repository grew out of an experiment in how to collaborate with ChatGPT to
build complete software systems.

Instead of asking ChatGPT for individual functions, snippets, or patches, the
experiment treated ChatGPT more like a **system compiler**. A complete software
system would be generated from a written specification. When the specification
changed, the entire system would be regenerated from scratch.

In this mode of working, the specification becomes the primary artefact. Code is
not edited directly. Progress comes from refining the specification, resolving
ambiguity, and making intent explicit, then asking ChatGPT to regenerate the whole
system again.

At first, this feels surprisingly effective. Large amounts of coherent code can be
produced quickly, and global consistency is often better than with incremental,
hand-written changes.

Then problems appear.

Specifications drift. Ambiguous statements are interpreted differently across
iterations. Constraints that were once explicit become implicit, then disappear.
The generated system still “works”, but no longer quite does what was originally
intended.

This repository exists because those failures were **instructive**.

---

## The Compiler Metaphor

The collaboration model captured here borrows deliberately from how compilers are
used.

A written specification plays the role of the program being compiled. ChatGPT plays
the role of a generator that emits a complete system. The system is regenerated in
full each time the specification changes. Problems in the specification must be
surfaced and resolved before generation can be trusted.

In this metaphor, ambiguity matters. An ambiguous specification is like a program
that compiles but whose meaning is unclear: it may produce output, but that output
cannot be relied upon. Where possible, ambiguity is treated as a kind of
compilation error that must be resolved explicitly rather than worked around.

The metaphor is imperfect, but it is a useful discipline.

---

## Why a Process Is Needed

Early iterations showed that simply “cleaning up” or “rewriting” specifications is
not enough.

One recurring experience was deeply unsettling. A small, local edit to the
specification — changing three words in a sentence, or rephrasing a paragraph for
clarity — would lead to a system that behaved quite differently in ways that felt
unrelated to the change. Nothing obviously broke. The system still regenerated
successfully. And yet, something fundamental had shifted.

From the human perspective, this feels wrong. We expect local changes to have local
effects. When they do not, confidence erodes quickly.

A useful way to understand what’s happening is through a familiar example.

If you ask a human partner to “buy some cake or chocolate”, most people will return
with one or the other. The unspoken default is an exclusive OR. A machine, given the
same instruction, will correctly interpret it as an inclusive OR and return with
both. Neither interpretation is unreasonable. The mistake is assuming they are the
same.

That example using OR vs XOR is from a real life, pre-AI project
where the difference in human vs machine interpretation of "or" cost a company
many thousands of pounds, and wasted several months work by a team. 

This exact mismatch shows up when regenerating systems from specifications. A small
wording change can quietly flip which interpretation of an implicit “or” is in play.
From the human perspective, three words changed and the system now behaves
differently for no obvious reason. From the generator’s perspective, the ambiguity
was finally resolved in a different, equally valid way.

These experiences reveal a deeper asymmetry. Humans perceive ambiguity as something
temporary or harmless — something that can be resolved socially, contextually, or
later. A generator must resolve ambiguity immediately and definitively. When intent
is not made explicit, the generator is forced to choose, and that choice **becomes part**
of the system’s behaviour.

Once this is understood, the failures stop being mysterious. They are not bugs in
the generator. They are failures of the specification to be a stable, authoritative
input.

What’s needed is a disciplined way to take evolving, human-written specifications
and transform them into a form where ambiguity, assumptions, and invariants are made
explicit — or are surfaced as problems — before regeneration occurs.

That disciplined transformation is what this repository calls **Specification
Canonicalisation**. Ps. We agree that is a terrible name - but for now it's the
most accurate name we can think of.

---

## What This Repository Is

This repository captures the **process, policies, and artefacts** required to make
compiler-style collaboration with ChatGPT reliable.

It is not just a project specification. It is a **process pack** that defines:

- how specifications are written and refined,
- how they are transformed into a canonical form,
- how the human in the loop collaborates with ChatGPT, and vice versa,
- how complete systems are generated from them,
- and how loss, ambiguity, and inconsistency are detected.

The pack includes both **reusable policy documents** and a **real system** that
serves as a concrete example of the process in practice.

---

## Want to See This Work?

If you’d prefer to see the process in action before reading further, this
repository includes a short **Quick Start**.

The quick start walks through generating a complete software system in a single,
compiler-style run using the specifications in this pack. It is intentionally
narrow in scope and exists to demonstrate that the process produces real,
self-contained systems.

You can find it in **quick-start.md**.

Reading it is optional. The rest of this README focuses on explaining the structure
and intent of the pack.

---

## A Concrete Example

This repository includes a real software system specification produced using the process it
describes.

If you want to ground the ideas here immediately, you can skim 
**system-behaviour.md**.
It defines the required runtime behaviour of a concrete system using the same
compiler-style collaboration and canonicalisation discipline described in this
README.

The details are not important at this stage. Its presence is meant to reassure:
this process is not theoretical, and it is designed to produce complete,
deployable systems.

---

## Reusable Policy vs Project Plug-Ins

The documents in this repository are deliberately split into two categories.

Some documents define general policy and are intended to be reused across projects.
Others define the details of a particular system and are swapped out when the
process is applied elsewhere.


The intervention specification (`interventions.md`) is a project-local artefact.
In other applications of this pack it MAY be empty, reduced, or replaced, depending
on what repeated regeneration reveals.

The reusable policy documents define:

- some words like COULD and SHOULD and their exact meaning in this context 
- how ChatGPT is expected to behave as a generator:
   - rules for generation
   - rules for human collaboration
   - rules for validation through generated tests
- and how specifications are canonicalised without losing meaning.

The project-specific documents define:

- the required runtime behaviour of a concrete system,
- and the tooling required to build and operate it.

In this repository, the concrete system exists primarily as a worked example and
validation target for the process itself.

---

## How to Read This Pack

If you are new to this repository, a suggested reading order is:

1. **README.md**  
   Orientation, motivation, and the overall structure of the pack.

2. **Reusable policy documents**  
   These define the collaboration and canonicalisation rules that apply across
   projects.

3. **Project-specific documents**  
   These define the concrete system used as an example plug-in.

---

## Scope and Non-Goals

This repository does not attempt to define a universal software methodology.

It documents one specific approach to human–LLM collaboration focused on:

- full regeneration rather than incremental patching,
- explicit contracts rather than inferred behaviour,
- and disciplined iteration over specifications rather than code.

Other approaches may be valid. This pack exists to make this one coherent,
inspectable, and shareable.
