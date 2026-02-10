# Lessons Learned

## key lessons attempting to use ChatGPT as a spec-driven codebase generator

## Orientation

This document records a set of key lessons learned while attempting to use ChatGPT as a spec-driven codebase generator — that is, to automatically generate the complete source code, tooling, scripts, and tests for a software system solely from an input specification.

The primary motivation for this approach is to liberate the specification author. By regenerating the codebase from scratch on each iteration, the usual notion of change at the implementation level disappears. The codebase is never modified; it is simply re-instantiated from the specification, which becomes the sole object of evolution.

It means the usual mental distinction between “small” and “large” changes disappears. Introducing substantial new behavior feels no different from adjusting a minor validation rule. In both cases, the interaction is the same—express the change in the specification and regenerate the codebase. The sense of risk that normally scales with the size of a change simply never arises.

Conceptually, this mirrors the way a programming language compiler operates: specifications take the place of source code, and the output is a complete, buildable codebase materialized as a zipped filesystem.

A key secondary benefit of this approach is global coherence. Incremental modification of previously generated code tends to accumulate inconsistencies, local fixes, and historical residue. Full regeneration avoids this entirely, ensuring that each iteration produces a fully coherent system as a single, unified act of design rather than a sequence of patches.

Over time, the specification pack itself evolves into a more generally useful and reusable artifact—capturing not just system behavior, but also constraints, conventions, and structural expectations that apply across a class of systems.

The specification pack is intended to be mostly generic. It is not tied to a particular system behavior or programming language, but instead aims to express broad software design principles at a level of abstraction suitable for reuse. That said, it does include an exemplar target system, and it has been developed primarily through experiments generating Go code.

The process began with deliberately minimal specifications. Each iteration exposed new failure modes in the generated output. Cases where the result would not compile, was logically wrong, incomplete, incoherent, or otherwise unfit for purpose. Rather than patching the specification to address each individual failure, I consistently stepped back to investigate the root cause that **made it possible** for the failure to occur.

In practice, the majority of these issues arose from systematic differences between how humans reason and how large language models reason. Particularly around implicit assumptions, abstraction boundaries, and unspoken conventions. For each such failure, ChatGPT and I explored how the underlying cause could be generalized and addressed at the highest possible level of abstraction, so that the resulting specification change would mitigate an entire class of related problems rather than a single instance.

The remainder of this document surveys the major categories of problems encountered and the strategies used to avoid them.

It concludes by examining the fundamental limitations uncovered by this approach, and how those limitations suggest modified approaches that more directly address the underlying constraints of current language models.

## Lessons Arising from Human vs. LLM Thinking Models

A recurring source of failure in early iterations was a mismatch between what the specification implicitly assumed and what the language model was entitled to infer. Many rules in the specification pack ultimately exist to bridge this gap between human intuition and LLM interpretation.

From a human perspective, it often feels sufficient to define the perimeter of authority. For example, the compiler-contract part of the specification states that “this set of specification files jointly define a single unified specification domain.” A human reader naturally interprets this as establishing a closed world: everything that matters is contained within those files, and nothing outside them should exert influence.

But that is not what the spec clause mandates. The model does not make that leap. From its perspective, declaring a unified domain does not preclude drawing on external conventions, implied best practices, or broadly learned norms—unless this exclusion is made explicit. In effect, the model reasons: you have told me how these files relate to each other, but not what I must ignore, nor how to behave when external knowledge appears to conflict with them.

This gap surfaced repeatedly in practice. The generated systems would subtly incorporate assumptions, idioms, or architectural patterns that were reasonable in general, but incorrect or actively harmful within the intended design space. Crucially, these influences were invisible to the human author, because they originated outside the written specification.

The specification pack therefore evolved to make **authority** explicit rather than assumed. Rules increasingly took the form of both positive and negative constraints: you must do X and you must not do Y. Modal language such as MUST, SHOULD, and LOCKED was introduced, followed by a dedicated compliance specification in the pack, that defines how these terms are to be interpreted and enforced. Similarly, explicit precedence rules were added to govern conflicts between written requirements and any inferred or external “best practice” the model might otherwise apply.

Viewed abstractly, these rules do not add new functional requirements. Instead, they serve to close the open world that the model naturally inhabits, replacing human intuition with machine-legible boundaries.

## The sequence of LLM thinking that generates an "answer"

Another class of problems arose not from what the model knows, but from when it commits to conclusions. By default, ChatGPT exhibits a strong tendency toward early synthesis: as soon as it has enough information to sketch a plausible solution, it begins constructing parts of the output (conceptually), filling gaps with assumptions as needed.

From the model’s perspective, this is an efficient and often successful strategy. In a conversational setting, early commitments can be revised or patched later with relatively little cost. In the context of whole-system code generation, however, this behavior is deeply destabilizing.

In practice, the model would make architectural decisions before the full constraint space had been established. Later-encountered requirements would invalidate those early choices, but rather than discarding them cleanly, the model would attempt to reconcile the inconsistency through local adjustments. The resulting codebases exhibited a characteristic kind of damage: orphaned abstractions, vestigial structures, and conceptual seams where incompatible decisions had been forced to coexist.

The specification pack gradually evolved to counteract this tendency by externalizing temporal (i.e. sequencing) structure. One early mitigation was the introduction of an explicit assumptions log, making provisional reasoning visible rather than implicit. This helped surface premature commitments, but did not prevent them.

Ultimately, it became necessary to constrain the order of reasoning itself. The specification began to require gated, sequential phases of thought, with certain categories of decision explicitly forbidden until earlier phases were complete. Analysis, commitment, and realization were separated, and the model was instructed to defer synthesis until the relevant constraint boundaries had been fully enumerated.

These measures do not make the model “more careful” in a human sense. Instead, they replace an internal, opaque reasoning sequence with an externally imposed one—trading flexibility for coherence in a context where coherence is paramount.

## The code works. But what would an experienced code reviewer object to?

A recurring form of resistance came not from failing code, but from code that worked while still provoking a strong negative reaction when viewed through a code-reviewer’s lens. These were cases where the generated system met the stated requirements, yet triggered the familiar human response of: this makes me uneasy, and I’m going to have to explain why.

The discomfort was rarely about correctness. Instead, it arose from patterns that an experienced reviewer would flag as fragile, obscure, unnecessarily clever, or misaligned with long-term maintainability. Explaining these objections to the model proved difficult, because they are often grounded in tacit professional judgment rather than explicit rules: an accumulation of lessons about what tends to break, what resists comprehension, and what quietly increases future cost.

One could reasonably argue that such concerns should not matter in a regenerate-each-time model. If the system is discarded and rebuilt on every iteration, why care about human readability or reviewer sensibilities at all?

In practice, they matter a great deal. For the foreseeable future, a human must inspect the generated output—sometimes lightly, sometimes deeply—on every iteration. Code-review instincts therefore act as an important quality filter, not because the code must be preserved, but because these instincts reliably identify structural weaknesses and conceptual debt that would otherwise go unnoticed. The value of providing influence over this class of problems in the specification pack is intrinsically multiplied over time, as the pack is reused on future projects. That is a big win.

The core misalignment here is subtle. The model optimizes for satisfying explicit requirements, whereas experienced reviewers apply an additional, largely implicit evaluation: would I trust this code if I had to live with it? That evaluation incorporates concerns about failure modes, cognitive load, and the ease with which future changes could be made safely—none of which are guaranteed to be captured by functional requirements alone.

To address this, the specification pack gradually absorbed a set of deliberately opinionated constraints, expressed at the highest viable level of abstraction. At the code level, these included discouraging (but not banning) unnecessary concurrency, avoiding overly complex or gratuitous regular expressions, and enforcing exhaustive error handling. At higher conceptual levels, they extended to enforcing system boundaries via interfaces, mandating consistent test composition patterns, clarifying the communicative role of tests for human readers, and strongly preferring system-level tests over fine-grained unit tests.

That last preference had downstream consequences. If behavior is primarily validated at system boundaries, then internal code paths must be reachable through externally exposed controls. This, in turn, influenced how interfaces were designed and how configuration “knobs” were surfaced. Importantly, these constraints were expressed in a programming-language-agnostic way, allowing them to shape structure without prescribing implementation details.

None of these rules make the model more capable of best-practice reasoning in any intrinsic sense. What they do instead is narrow the available choice space, biasing generation toward designs that align more closely with experienced human judgment. In effect, they encode reviewer sensibilities as environmental constraints — substituting explicit guidance for what would otherwise remain an invisible and unmet expectation.


## Making life easier for the human

In practice, spec-driven code generation often produced codebases that were harder for me to assess than they needed to be. The issue was not correctness per se, but the effort and time required for me to evaluate the generated codebase at each iteration.

Over time, I became increasingly focused on reducing that assessment cost. What I wanted were early, decisive signals: a way to surface blocking problems almost immediately, react to the first one encountered, and ignore everything else until that issue was resolved by updating the spec. Fixing the first blocker would define the next specification and regeneration cycle.

Part of this was a methodological shift on my part, but that methodology could only work if the generated code supported it. That, in turn, required introducing new constraints into the specification about how code should be generated, not just what it should do.

A central example was testing. I wanted the generated tests to validate the entire exposed behavior of the system, rather than focusing narrowly on internal units. This requirement had far-reaching consequences for the specification. It forced clearer separations of concern, explicit definition of public behavior surfaces, and systematic use of dependency injection for external services—using patterns that made deterministic fake implementations straightforward to generate.

It also imposed a further constraint: internal business logic branches needed to be reachable through explicit inputs, so that system-level tests could exercise them directly. That requirement propagated upward through the call structure, shaping interfaces and control flow to make behavior observable and testable from the outside.

In the generated codebase, this ultimately manifested in a very simple hook: a Makefile with a single `test` target. My first interaction with any newly generated codebase was always the same; to run `make test`. (In reality I also had to precede that with `go get .` to resolve and fetch the dependencies - but I didn't want to clutter the message).

That one command answered the most important initial questions immediately. Did it compile? Did any tests fail? If it didn't compile, or any test failed, I treated that failure as the next root cause to investigate, following it down until it revealed a conceptual mistake in the specification or a systematic failure mode in the model’s reasoning. Fixing that root cause in the specs then defined the next iteration.

If all tests passed, I scanned them to see whether the coverage and intent looked broadly sensible. If that also checked out, I moved on to the generated assumptions and deviation logs. Any red flags there became the next focus. And so the loop continued.

Another guideline I included in the specs, is how comments in the code should be generated conceptually in a way that helps me to scan-read the codebase to absorb the big picture rapidly. I asked it to summarise each function and non trivial type in such a way that I could read the comment instead of the code when that would better serve this type of surface traversal.

## Programming language specific problems

The specification pack allows not only the description of required system behavior, but also the imposition of constraints on platform, architecture, deployment model, and implementation language. In practice, a particular programming language is usually mandated, though conceptually this choice is intended to function as a plug-in rather than a defining feature.

In the exemplar specification pack, the system is defined as a REST API for receiving, persisting, and analyzing telemetry events. The architecture is constrained to a deployable Google Cloud Function, with Google Cloud Storage used for persistence, and Go (version 1.25) specified as the implementation language.

Across successive generation runs, a recurring set of problems appeared that initially seemed specific to Go. Over time, it became clear that while these issues were surfaced through the Go ecosystem, many of their root causes could be addressed through language-independent constraints in the specification.

One of the most persistent difficulties concerned the use of third-party packages. The model oscillated unpredictably between excessive reliance on external libraries and avoiding them altogether. Neither extreme was acceptable. Attempts to express taste-based or policy-style guidance when a dependency was “worth it” proved ineffective. I was unable to encode my judgment in a form the model could reliably follow.

The eventual approach was a hybrid. In a small number of cases, I enforced what felt like unambiguous decisions: for example, using the official Google Go SDK to integrate with Google Cloud Storage, and using a well-known external validator for JSON payloads. Even this was surprisingly difficult to get right. In one iteration, the model complied literally with the requirement to use a validator, but integrated it in a way that performed no meaningful validation at all. It was there in plain sight in the code but making no logical contribution whatsover - i.e. the real validation was done outside of it. 

Ironically, the most robust generated solution turned out to be expressing the validation logic directly, without a third-party package. Done carefully, this produced clearer, more reliable code, but only provided that other constraints were in place to prevent the generation of overly complex or fragile regular expressions. This episode became a useful illustration of a broader pattern: mandating what to use is often less effective than constraining how behavior should be expressed.

Another major class of problems arose around dependency management. Go provides a simple and well-defined mechanism for declaring and resolving dependencies, but in practice this workflow depends on tool-driven discovery and network access. The model, lacking reliable access to the package ecosystem, repeatedly attempted to fully materialize dependency metadata. Often inventing non existent versions or made-up checksums in the process.

Attempts to tell the model to “leave dependency resolution incomplete” frequently backfired, causing it to avoid external dependencies entirely. The workable solution was to require best-effort declaration of direct dependencies only, with the understanding that a human would complete resolution after generation. This assumption is trivial for a person, but difficult for the model to reason about without explicit guidance.

A further class of language-specific problems emerged when the model was forced to reason about multiple syntaxes at once. A common example was generating Go string literals that embed JSON. The model struggled to satisfy the syntactic and escaping rules of both languages simultaneously, leading to brittle or invalid output. This failure mode generalized cleanly: wherever possible, the specification was updated to prohibit embedding fragments of one language inside another, preferring structural representations instead. The same rule applies equally to embedded SQL or other DSLs.

Taken together, these experiences reinforced a recurring theme: many problems that appear language-specific are better understood as symptoms of deeper constraints in how the model reasons about structure, tooling, and composition. Addressing them effectively required elevating the solution into the specification, rather than treating them as quirks of a particular language.


## Syntanctial generation mistakes
- languages inside languages
- runes
- non printable characters

## test fixtures overreaching