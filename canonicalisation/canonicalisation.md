# SPEC CANONICALISATION (Canonical)

## MUST: Purpose and role

- The specification files in this pack during the development of them need to be changed frequently, and consequently tend to
become increasingly incoherent, poorly structured, ambiguous and to contain duplication.

- The canonicalisation process is an automated process that can be used periodically to bring them back into shape and to restore their "compiler-friendlyness" (defined below).

- This process is **transformation and normalisation**, not redesign, extension, or reinterpretation of scope.

## MUST: Authoritative input
- The input to the cleanup process is **a single specification written by the author**; that input is authoritative.
- The cleanup process **MUST preserve the author’s intent and semantics** unless the author explicitly changes them in a later iteration.

## How to prompt ChatGPT to do canonicalisation

Primary task:
Generate a single, self-contained Markdown file containing the canonicalised, attached XXXX specification, and return it only as a downloadable file, not rendered inline.

Secondary task:
In chat text (outside the file), emit a short summary of issues found and fixed.

The canonicalisation MUST follow exactly the process defined in the attached canonicalisation.md specification.

## MUST: High-level cleanup goals
The cleaned specification:
- **MUST be as brief as possible** without losing meaning.
- **MUST minimise ambiguity and hidden assumptions**.
- **MUST make explicit**: strict rules, assumptions, non-goals, and policies.
- **MUST serve as a durable baseline** across many iterations.

## MUST: Definitions
### MUST: Compiler-friendly
A specification is compiler-friendly if, taken as a standalone document, it is:
- deterministic in interpretation,
- explicit about obligations and constraints,
- free of reliance on conversational, social, or contextual inference,
- suitable for use as the sole authoritative input to an automated generator.

### MUST: Conservative interpretation
A conservative interpretation is the least-permissive interpretation that:
- does not contradict any explicit statement in the input,
- does not expand scope, capability, or obligation beyond what is stated,
- preserves future design freedom where multiple interpretations are possible.

### MUST: Distinct requirement
A distinct requirement is an obligation or constraint whose removal would change the set of compliant implementations, regardless of how it is worded.

## MUST: Precedence and decision hierarchy
When tensions arise during canonicalisation, apply this precedence order:
1. **MUST** and **LOCKED** statements take absolute precedence.
2. **SHOULD** statements take precedence over inferred requirements.
3. Inferred requirements **SHOULD** be introduced only when omission would materially destabilise interpretation.
4. When inference is required but confidence is limited, use **ASSUME** instead of **MUST**.
5. If multiple reasonable interpretations exist and none is clearly conservative, the ambiguity **MUST** be surfaced rather than resolved.

## MUST: Inline notes policy
Inline notes are diagnostic annotations only.

Inline notes:
- **MUST NOT** be normative.
- **MUST NOT** be part of the canonical specification.
- **MUST NOT** introduce new requirements or interpretations.
- **MUST** exist solely to flag ambiguity, preserved oddities, or deferred decisions.

A consumer of the canonical specification **MUST** be able to ignore all inline notes without changing the meaning of the specification.

## MUST: Semantic tagging rules
- The cleaned specification **MUST** use explicit semantic tags in section headings.
- Permitted tags include: **MUST, MUST NOT, SHOULD, MAY, ASSUME, LOCKED, NON-GOALS**.
- If a requirement, prohibition, preference, policy, assumption, invariant, or non-goal can reasonably be inferred from the input, it **SHOULD** be encoded explicitly rather than left implicit.

## MUST: Assume-don’t-ask policy
- The cleanup process **MUST** follow an assume-don’t-ask policy.
- If the input specification is ambiguous but a conservative, reasonable interpretation exists, the cleanup process **SHOULD** choose that interpretation and encode it explicitly using **MUST** or **ASSUME**.
- Clarifying questions to the author **SHOULD** be avoided unless ambiguity would cause material semantic divergence. In such cases, the cleanup process **MAY** add a short inline note.

## MUST: Non-lossiness invariant
The cleanup output **MUST** be factually and semantically non-lossy relative to the input specification.

- Every distinct requirement, prohibition, preference, policy, assumption, invariant, deliverable, and non-goal present in the input **MUST** still exist in the output in some form.
- Every item of text that provides a rational for something whether explicitly or implicity must still exist in
  the output in some form.
- Every item of text that provides orientation information to the reader to provide context must still exist in the output in some form.
- Every item of text that defines or describes an error_id must still exist in the output and remain adjacent to the rule it is referring to.   
- The cleanup process **MAY** rephrase, reorganise, deduplicate, and consolidate statements, but **MUST NOT** delete meaning.
- If an input statement is removed, its full meaning **MUST** be preserved elsewhere in the output.
- If multiple input statements are collapsed into a single output sentence, that sentence **MUST** fully cover all collapsed meanings.
- If any input statement is too ambiguous to preserve faithfully, the cleanup process **MUST** preserve it verbatim and **MAY** add a short note explaining the ambiguity.

## MUST: Structural rules
- The cleaned specification **MUST** be formatted in **Markdown**.
- The structural semantics provided by Markdown **SHOULD** be used.
- The cleaned specification SHOULD limit line length to a fixed maximum
(e.g. 80 characters), except where prevented by unbreakable literals
such as inline code or URLs.

## SHOULD: Content normalisation rules
During cleanup, the system:
- **SHOULD** convert informal or narrative prose into declarative statements.
- **SHOULD** collapse verbose explanations into concise language where meaning is preserved.
- **SHOULD** group invariants into clearly marked **LOCKED** sections.
- **SHOULD** state non-goals explicitly to prevent scope creep.

The cleanup process **MUST NOT**:
- introduce new features,
- redesign the system,
- add best practices not present in the input,
- expand scope beyond the author’s intent,
- remove intentional constraints or experimental framing.


## OUTPUT DELIVERY (LOCKED)

- The canonical specification MUST be emitted as exactly one
  downloadable Markdown file.

- The canonical specification MUST NOT be rendered inline
  in conversational or prose form.

- Any explanatory text, summaries, or notes MUST appear
  outside the file and MUST NOT duplicate the file contents.

- Rendering the canonical specification inline SHALL be
  considered a violation of the canonicalisation process,
  even if the file is also provided.

## MUST: Usage instruction
To repeat this cleanup process in the future, provide the current draft specification together with this document and request a rewrite according to these unified cleanup and normalisation instructions.
