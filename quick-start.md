# Quick Start

This document is a **demonstration**, not a replacement for the rest of the pack.

Its purpose is to show, as directly as possible, what this collaboration model
enables: generating a complete software system from specification in a single,
compiler-style step.

If you want to understand *why* this works and *when* it does not, start with
README.md. If you want to *see it happen*, continue here.

---

## What This Demonstrates

Using the documents in this repository, you can ask ChatGPT to act as a **system
compiler** and generate a complete, self-contained software system from scratch.

In one prompt, ChatGPT will:

- treat the specifications you provide as authoritative input,
- generate a full source tree implementing the defined system,
- include all required components, configuration, and structure,
- and return the result as a coherent repository you can download and inspect.

This is not a toy example. The system specification included here is real, and the
generated output is intended to be complete.

---

## What This Is (and Is Not)

This quick start is intentionally narrow in scope.

It **is**:
- a concrete demonstration of compiler-style generation,
- a way to verify that the process produces real systems,
- a baseline from which further iteration can begin.

It **is not**:
- a guarantee of correctness in all environments,
- a substitute for specification canonicalisation,
- or a shortcut around understanding the policies in this pack.

Think of this as a “hello, world” for full-system generation.

---

## Providing the Specification to ChatGPT

ChatGPT does not have access to this repository by default.

Before running the generation prompt, upload the specification documents from this
repository into the ChatGPT conversation. Treat this as supplying the complete
compilation unit.

The compilation input is **exactly** the set of specification files in this
repository. No more and no less. Nothing outside the uploaded files should be
assumed to exist, and nothing within them should be omitted.

---

## The One-Shot Generation Prompt

To run the demonstration, open a fresh ChatGPT conversation and provide a prompt
along the following lines.

You may copy and paste this verbatim, adjusting only filenames if needed.

---

** ChatGPT Prompt:**

- This is an instruction to the human.
- Making ChatGPT follow the required process has been found to be very fragile
- So we suggest you use the exact prompt below

```
You are acting as a system compiler.
You are authorized to read and extract files from the attached zip file.
Open compiler-contract.md from that zip. First output your Phase 0–3 assimilation summary as defined there, then continue exactly as specified.
```

---

## About Generation Time

Full-system generation is intentionally heavyweight.

Depending on system size and complexity, it is normal for a generation run to take
several minutes from submitting the prompt to receiving the final, downloadable
output. Five minutes or more is not unusual.

This delay is a feature of the model being asked to reason globally about an entire
system, not a sign of failure or inefficiency. Treat generation as a deliberate,
batch-style operation rather than an interactive exchange.

---

## What You Should Expect to See

If the process is working as intended, the output you receive should:

- look like a real software repository,
- contain multiple files organised by responsibility,
- reflect the structure and constraints described in SYSTEM_SPEC.md,
- and be internally consistent without manual patching.

You should *not* expect the output to be minimal, optimised, or stylistically
polished. The goal is correctness relative to the specification, not elegance.

---

## What to Do Next

After running the quick start, you have several options:

- Inspect the generated code to understand how the specification was interpreted.
- Run or deploy the system, if applicable, to validate behaviour.
- Return to README.md to understand the broader process and its constraints.
- Begin an iteration by refining the specification and regenerating the system.

The rest of the pack exists to make those next steps reliable.

---

## A Final Note

If this quick start feels surprisingly powerful, that reaction is intentional.

The rest of the documents in this repository exist to answer the harder question:
how to make this power predictable, repeatable, and safe to use over time.
