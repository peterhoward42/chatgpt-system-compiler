
## Collaborating with ChatGPT as a System Compiler

This repository grew out of an experiment in trying to make ChatGPT generate a
complete software system from a specification (spec).

- Single shot. Fully correct. With good test coverage. Embodying excellent engineering quality best practices.
- Reusable across projects
- You provide the following (conceptually) as plugins:
  - The desired system behaviour contract
  - Programming language to use and associated rules (if language-specific rules become necessary)
  - Architecture and deployment rules
- Delivers the resultant code base as a zipped, fully formed file system (a repo). 

To use it, you iterate your spec not the code - and trigger a complete regeneration

## How I designed and orchestrated the experiment

- Start with a deliberately naive system behaviour spec and nothing else
- Generate the code base to reveal one systemic flaw at a time
- Work backwards from the found fault - in conversation with ChatGPT - to learn how and why an LLM could make this mistake
- Work out in collaboration with ChatGPT a way to express a rule to prevent this mistake
- Upgrade the rule to the the highest possible level of abstraction we could find - to make it cover a wider family of similar flaws
- Put that new rule in the spec
- Iterate

## Was the experiment successful?

- In many ways yes. It got to a point with the built-in example system spec that it produced a code base I was completely satisfied with in regard to correctness, high engineering quality, and test precision.
- A large proportion of the specs (a set of files) is encouragingly project-agnostic and should be reusable to a large degree.
- Crucially though - even two consecutive generations from the same spec produced code bases that can be significantly different from each other.
- That non deterministic behaviour is very hard to ajust to psychologically
- It also means that one's judgement of it being able to do an adequate job is fundamentally probabilistic - which again is hard to adjust to.
- This first phase of the experiment ended however with a bit of an unpleasant bump. The experimental trajectory produced better and better results as it proceeded. Until it didn't.
- Eventually I learned (in collaboration with ChatGPT) that we'd reached a hidden contextual memory limit, with very bad and destructive manners.
- At that point it decided internally and silently that the best way to proceed was to make a private judgement about which of the rules it could quitely forget about. Obviously it started regressing in output quality very badly at that point - and very randomly and confusingly too. The lessons-learned.md document suggests some ways forward to overcome this problem.

## A choice of next steps for you

- Give it a quick whirl - no theory - just try it - see quick-start.md
- Read system-behaviour.md - to assimilate the built-in example system it's trying to generate 
- Read the detailed lessons-learned document in lessons-learned.md
- Browse all the other files - which jointly form the spec pack - they are self documenting
