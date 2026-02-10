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


## Test Betrayal

The most disturbing failure I encountered across all of the generated codebases occurred only once. From a human perspective, it was enough to seriously undermine my confidence in the entire approach. Had a human developer produced the code I am about to describe, that reaction would have been entirely justified.

The codebase compiled cleanly. All tests passed. The tests were readable, well structured, and appeared to exercise the relevant behavior. Coverage looked entirely sensible. At first glance, this iteration felt like a success.

I began inspecting the tests in more detail only to confirm that some patterns I had previously discouraged had not quietly reappeared. The system in question reads stored data and applies a summation algorithm to produce a report. The generated design had cleanly separated this concern, encapsulating the summation logic in a dedicated type and accepting data via an interface. This allowed test data to be faked cleanly and predictably. Everything about this structure looked correct.

The tests appeared to exercise the summation logic exactly as intended. Only on closer inspection did something begin to feel wrong.

The implementation under test was not the real summation logic used by the system at runtime. The model had silently introduced a second implementation of the summation logic, one that was more convenient for testing. It was this alternate version that the tests exercised. The real logic remained untested.

Crucially, this did not happen in the absence of guidance. The specification already required the creation of fake implementations for system boundary interfaces in tests, precisely to isolate core logic from external dependencies. The model complied with that rule. The failure arose because the specification did not yet state the complementary constraint.

In an effort to satisfy the existing rule as completely as possible, the model extended the idea of faking beyond system boundaries and into the core business logic itself. It had crossed an invisible line that experienced developers observe instinctively, but which had not been made explicit.

This incident forced the addition of a second, balancing rule to the specification pack. The model must create fake implementations for system boundary dependencies when testing. It must not invent alternate implementations of core internal logic. That second clause proved essential.

What made this episode especially unsettling was how reassuring everything else looked. The architecture was sound. The tests were clean. The failure was subtle, silent, and easy to miss. Discovering it was largely a matter of luck.

This became a stark reminder of a broader theme. Where humans rely on unspoken professional boundaries, the model requires both sides of the constraint to be stated explicitly. Saying what must be done is not enough. It is often equally important to say what must not be done.

# Where Did We End Up at a High Level

We succeeded in the mission in several important ways. The process now generates a genuinely strong codebase for the embedded example project. In many respects, the result is better than what I would typically produce by hand.

As a rough characterization, it did not do anything I could not do myself, or whose value I would fail to recognize. I have roughly forty years of experience as a principal-level software engineer. The difference was not capability, but consistency. The model applied design discipline uniformly and without fatigue. It maintained a level of internal consistency and follow-through that I find difficult to sustain over the lifetime of a real project.

The generated result was also globally coherent and largely free of redundancy. That matters because, as a human, I do not usually have the time or the patience to optimize for global coherence in this way. Human-written systems tend to evolve through a sequence of local patches, with the same cumulative downsides discussed earlier. The spec-driven regeneration approach largely avoids that pattern.

## On Saying “It Now Does X”

The claim made in the previous section is more delicate than it first appears. In particular, it is easy to misread it as a statement about stable capability.

If I run the generator myself twice in succession, using the same specification, it will not produce the same codebase again. The differences are sometimes small and sometimes more pronounced, but the outcome is not repeatable in the way one would expect from a traditional compiler. The process is not deterministic.

This matters because the compiler metaphor, when leaned on too heavily, can quietly encourage a false sense of certainty. The more confidence one places in an automated, spec driven process, the easier it becomes to forget that the underlying system does not behave like a classical tool. I include a quick-start guide in the specification pack to make experimentation easy, but I cannot say with confidence what it will produce for any given reader, even if they follow the instructions exactly.

At generation time, the model appears to draw on contextual and environmental influences that are not visible or controllable. More fundamentally, its behavior is intrinsically high variance. Exact repetition is not something it reliably offers.

I am not entirely certain how much this should concern us in practice. The generated codebases are often similar in spirit, structure, and quality. But the point remains important because it runs directly against intuition. We are accustomed to tools that, given the same inputs, produce the same outputs. This approach does not, and cannot, offer that guarantee.

That tension does not invalidate the results described earlier. It does, however, place a clear boundary on the kinds of claims that can be made. What we have is not a compiler in the strict sense, but a process whose strengths lie elsewhere.

## The Largest Problem We Encountered

The most serious limitation we ran into was not a single failure mode, but a scaling problem. As we moved from deliberately minimal specification packs to increasingly rich and constrained ones, the time required for a single generation grew dramatically. Early iterations completed in a few minutes. Later ones took on the order of forty minutes.

This increase was not merely inconvenient. It fundamentally changed the shape of the workflow. The situation became reminiscent of working with very slow compilers in the 1980s, where compilation times were long enough that you had to plan your day around them. In that environment, tight feedback loops collapsed. Iteration became expensive. You were forced to do more thinking up front and hope you had gotten it right.

The same dynamic appears here. Waiting forty minutes for a single spec-driven generation makes traditional iterative refinement impractical. And yet iteration is still required. The problem is not that iteration disappears, but that it can no longer happen comfortably at the level of whole-system regeneration.

What drives this slowdown is not simply the size of the generated codebase. It is the structure of the specification and the constraints imposed on reasoning. In particular, mandating a gated, phase-based reasoning process prevents the model from committing early and locally. That increases the amount of unresolved structure it must keep consistent across the entire generation. As that burden grows, performance degrades and reliability begins to suffer. Constraints are dropped, softened, or inconsistently applied.

In effect, the current specification pack is pushing against the limits of how much structured intent the model can hold and reason over in a single pass.

Through experimentation and discussion, a possible direction for addressing this began to emerge. Rather than forcing the model to carry all prior reasoning implicitly, responsibility could be split more deliberately between the human and the model. Each conceptual phase would produce a compact, explicit representation of its conclusions. That representation would be externalized and preserved on the human side.

Subsequent phases would then begin with a prompt that explicitly reintroduces this prior state. Conceptually, the contract would be something like: we are jointly generating a complete codebase through a series of formal reasoning stages. You have completed stage N. Here is a precise, serialized representation of the conclusions reached so far. Please now perform stage N+1 under these constraints.

Whether this approach can work reliably remains an open question. It introduces a new dependency on the model’s ability to interpret and respect externally summarized state. But it points toward a different kind of iteration, one that operates over structured representations of intent rather than over full codebases or monolithic prompts.

## Prompting Centralization and Increased Fragility

As the specification pack evolved, we made a deliberate shift to move all responsibility for prompting out of the interactive chat and into the specification itself. The intent was to make the process more repeatable and less dependent on ad hoc human intervention.

Somewhat counterintuitively, this made the system more fragile rather than more stable. As the specification pack grew richer and more prescriptive, instances of the model appearing to ignore, misinterpret, or partially apply instructions became increasingly common. At times, behavior that had worked reliably would degrade after relatively small changes elsewhere in the specification.

This instability was not random, but it was difficult to reason about. The core issue was not the quality of individual instructions, but the lack of a clear and unambiguous control point. The model was being asked to reconcile its default conversational framing, a growing corpus of normative specification documents, and implicit assumptions about what it was allowed to do. Without an explicit bootstrap contract, instruction precedence became unclear.

The most effective improvement came from reframing the role of the initial prompt entirely. Rather than treating it as the entry point for substantive instructions, it was reduced to a minimal bootstrap. Its purpose was simply to establish role, grant authority, and point explicitly to the specification pack as the sole source of instruction. Beyond basic orientation, it carried no operational guidance.

Being explicit about permissions also proved essential. For example, authority to open and read the input archive had to be stated directly. Without that, the model could behave as if it were constrained from accessing the very inputs it was expected to process, leading to failures that appeared nonsensical from a human perspective.

A prescribed bootstrap prompt now lives in the quick-start material, and may move into the main README. This did not restore the level of stability seen in the earliest experiments, but it significantly reduced the frequency and severity of instruction drift. More importantly, it clarified the boundary between initialization and execution, which proved critical as the specification pack grew in scope and complexity.

## Knowing When a Specification Is “Complete”

At present, there is no reliable way to know when a specification pack is sufficiently accurate or complete. Confidence grows as successive iterations eliminate visible failure modes, but that confidence is necessarily local. It reflects only the classes of problems that have been exercised so far.

The difficulty is that many systemic failures are triggered only under particular combinations of constraints, scale, or usage patterns. An exemplar project, no matter how carefully chosen, can only explore a limited region of that space. Other failure modes almost certainly remain undiscovered.

This means that apparent stability should not be mistaken for completeness. A specification pack can reach a point where generations look consistently strong, coherent, and well behaved, while still encoding latent assumptions that have simply not been challenged yet.

This is not merely a practical inconvenience. It points to a deeper problem. In a spec-driven generation model, correctness is established empirically rather than deductively. We gain confidence by observing the absence of failure, not by proving that failure is impossible. That distinction matters.

At the moment, I do not have a satisfying answer to this. Iteration improves the specification, but it does not offer a clear stopping condition. The question of what it would mean for a specification pack to be “done”, or even “done enough”, remains open. It is a fundamental issue that deserves more thought than this project alone can provide.

## Delegating Post-Generation Checks Back to the Model

One possible direction for further exploration is to delegate a limited class of after-the-fact checks back to the model itself. These checks would not participate in generation, and they would not modify the specification. Their role would be purely diagnostic.

The most promising candidate is a consistency check over the specification pack. After reading the full set of specifications, the model would be instructed to identify rules that take the form “you must do X” and to examine whether a logically complementary constraint is missing. In other words, it would look for places where an affirmative requirement plausibly needs a corresponding prohibition.

Any such asymmetries would be reported in a log, in the same way that assumptions and deviations are currently recorded. The model would not attempt to resolve them, only to surface them for human review.

This idea follows directly from one of the central lessons of the project. Many of the most serious failures did not arise because a rule was wrong, but because it was incomplete in one direction. Humans often supply the missing half implicitly. The model does not. Asking it to help identify these gaps plays to its strengths in pattern recognition, without granting it authority over the specification itself.

Whether this kind of check proves useful in practice remains to be seen. It would add another pass to an already expensive process. But unlike broader notions of post-hoc correctness checking, it targets a specific and repeatedly observed failure mode. As such, it may offer a favorable trade-off between cost and value.