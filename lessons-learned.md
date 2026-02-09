# Lessons Learned

## Orientation

This document records a set of key lessons learned while attempting to use ChatGPT as a spec-driven codebase generator — that is, to automatically generate the complete source code, tooling, scripts, and tests for a software system solely from an input specification.

The primary motivation for this approach is to liberate the specification author. By regenerating the codebase from scratch on each iteration, the usual notion of change at the implementation level disappears. The codebase is never modified; it is simply re-instantiated from the specification, which becomes the sole object of evolution.

For the specification author, this has an unexpected consequence: the usual distinction between “small” and “large” changes disappears. Introducing substantial new behavior feels no different from adjusting a minor validation rule. In both cases, the interaction is the same—express the change in the specification and regenerate the codebase. The sense of risk that normally scales with the size of a change simply never arises.

Conceptually, this mirrors the way a programming language compiler operates: specifications take the place of source code, and the output is a complete, buildable codebase materialized as a zipped filesystem.

A key secondary benefit of this approach is global coherence. Incremental modification of previously generated code tends to accumulate inconsistencies, local fixes, and historical residue. Full regeneration avoids this entirely, ensuring that each iteration reflects the specification as a single, unified act of design rather than a sequence of patches.

Over time, the specification pack itself evolves into a more generally useful and reusable artifact—capturing not just system behavior, but also constraints, conventions, and structural expectations that apply across a class of systems.

The specification pack is intended to be mostly generic. It is not tied to a particular system behavior or programming language, but instead aims to express broad software design principles at a level of abstraction suitable for reuse. That said, it does include an exemplar target system, and it has been developed primarily through experiments generating Go code.

The process began with deliberately minimal specifications. Each iteration exposed new failure modes in the generated output—cases where the result was incomplete, incoherent, or otherwise unfit for purpose. Rather than patching the specification to address each individual failure, I consistently stepped back to investigate the root cause that allowed the failure to occur.

In practice, the majority of these issues arose from systematic differences between how humans reason and how large language models reason—particularly around implicit assumptions, abstraction boundaries, and unspoken conventions. For each such failure, ChatGPT and I explored how the underlying cause could be generalized and addressed at the highest possible level of abstraction, so that the resulting specification change would mitigate an entire class of related problems rather than a single instance.

The remainder of this document surveys the major categories of problems encountered and the strategies used to avoid them.

It concludes by examining the fundamental limitations uncovered by this approach, and how those limitations suggest future directions that more directly address the underlying constraints of current language models.


## Lessons Arising from Human vs. LLM Thinking Models

A recurring source of failure in early iterations was a mismatch between what the specification implicitly assumed and what the language model was entitled to infer. Many rules in the specification pack ultimately exist to bridge this gap between human intuition and LLM interpretation.

From a human perspective, it often feels sufficient to define the perimeter of authority. For example, the compiler-contract specification states that “this set of specification files jointly define a single unified specification domain.” A human reader naturally interprets this as establishing a closed world: everything that matters is contained within those files, and nothing outside them should exert influence.

The model does not make that leap. From its perspective, declaring a unified domain does not preclude drawing on external conventions, implied best practices, or broadly learned norms—unless this exclusion is made explicit. In effect, the model reasons: you have told me how these files relate to each other, but not what I must ignore, nor how to behave when external knowledge appears to conflict with them.

This gap surfaced repeatedly in practice. The generated systems would subtly incorporate assumptions, idioms, or architectural patterns that were reasonable in general, but incorrect—or actively harmful—within the intended design space. Crucially, these influences were invisible to the human author, because they originated outside the written specification.

The specification pack therefore evolved to make authority explicit rather than assumed. Rules increasingly took the form of both positive and negative constraints: you must do X and you must not do Y. Modal language such as MUST, SHOULD, and LOCKED was introduced, followed by a dedicated compliance specification that defines how these terms are to be interpreted and enforced. Similarly, explicit precedence rules were added to govern conflicts between written requirements and any inferred or external “best practice” the model might otherwise apply.

Viewed abstractly, these rules do not add new functional requirements. Instead, they serve to close the open world that the model naturally inhabits, replacing human intuition with machine-legible boundaries.

## The sequence of LLM thinking that generates an "answer"

Another class of problems arose not from what the model knows, but from when it commits to conclusions. By default, ChatGPT exhibits a strong tendency toward early synthesis: as soon as it has enough information to sketch a plausible solution, it begins constructing parts of the output, filling gaps with assumptions as needed.

From the model’s perspective, this is an efficient and often successful strategy. In a conversational setting, early commitments can be revised or patched later with relatively little cost. In the context of whole-system code generation, however, this behavior is deeply destabilizing.

In practice, the model would make architectural decisions before the full constraint space had been established. Later-encountered requirements would invalidate those early choices, but rather than discarding them cleanly, the model would attempt to reconcile the inconsistency through local adjustments. The resulting codebases exhibited a characteristic kind of damage: orphaned abstractions, vestigial structures, and conceptual seams where incompatible decisions had been forced to coexist.

The specification pack gradually evolved to counteract this tendency by externalizing temporal structure. One early mitigation was the introduction of an explicit assumptions log, making provisional reasoning visible rather than implicit. This helped surface premature commitments, but did not prevent them.

Ultimately, it became necessary to constrain the order of reasoning itself. The specification began to require gated, sequential phases of thought, with certain categories of decision explicitly forbidden until earlier phases were complete. Analysis, commitment, and realization were separated, and the model was instructed to defer synthesis until the relevant constraint boundaries had been fully enumerated.

These measures do not make the model “more careful” in a human sense. Instead, they replace an internal, opaque reasoning sequence with an externally imposed one—trading flexibility for coherence in a context where coherence is paramount.

## It works, but what would an experienced code reviewer object to?

A recurring form of resistance came not from failing code, but from code that worked while still provoking a strong negative reaction when viewed through a code-reviewer’s lens. These were cases where the generated system met the stated requirements, yet triggered the familiar human response of: this makes me uneasy, and I’m going to have to explain why.

The discomfort was rarely about correctness. Instead, it arose from patterns that an experienced reviewer would flag as fragile, obscure, unnecessarily clever, or misaligned with long-term maintainability. Explaining these objections to the model proved difficult, because they are often grounded in tacit professional judgment rather than explicit rules: an accumulation of lessons about what tends to break, what resists comprehension, and what quietly increases future cost.

One could reasonably argue that such concerns should not matter in a regenerate-each-time model. If the system is discarded and rebuilt on every iteration, why care about human readability or reviewer sensibilities at all?

In practice, they matter a great deal. For the foreseeable future, a human must inspect the generated output—sometimes lightly, sometimes deeply—on every iteration. Code-review instincts therefore act as an important quality filter, not because the code must be preserved, but because these instincts reliably identify structural weaknesses and conceptual debt that would otherwise go unnoticed.

The core misalignment here is subtle. The model optimizes for satisfying explicit requirements, whereas experienced reviewers apply an additional, largely implicit evaluation: would I trust this code if I had to live with it? That evaluation incorporates concerns about failure modes, cognitive load, and the ease with which future changes could be made safely—none of which are guaranteed to be captured by functional requirements alone.

To address this, the specification pack gradually absorbed a set of deliberately opinionated constraints, expressed at the highest viable level of abstraction. At the code level, these included discouraging (but not banning) unnecessary concurrency, avoiding overly complex or gratuitous regular expressions, and enforcing exhaustive error handling. At higher conceptual levels, they extended to enforcing system boundaries via interfaces, mandating consistent test composition patterns, clarifying the communicative role of tests for human readers, and strongly preferring system-level tests over fine-grained unit tests.

That last preference had downstream consequences. If behavior is primarily validated at system boundaries, then internal code paths must be reachable through externally exposed controls. This, in turn, influenced how interfaces were designed and how configuration “knobs” were surfaced. Importantly, these constraints were expressed in a language-agnostic way, allowing them to shape structure without prescribing implementation details.

None of these rules make the model more capable of best-practice reasoning in any intrinsic sense. What they do instead is narrow the available choice space, biasing generation toward designs that align more closely with experienced human judgment. In effect, they encode reviewer sensibilities as environmental constraints — substituting explicit guidance for what would otherwise remain an invisible and unmet expectation.


## Tricks to make my iteration reviewer's job easier
coming soon
- mention early focus on separation of concerns, strict external interface coupling, internal decoupling and genereated test coverage and test anatomy contraints
- then 1) does it compile - reject and investigate go no further. 2) do the tests run, 3) do the tests pass, 4) are there red flags in assumptions or deviation log? - only now dig deeper and wider.



## Syntanctial generation mistakes
- languages inside languages
- runes
- non printable characters