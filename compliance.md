# COMPLIANCE SPECIFICATION

o  COMPLIANCE WITH RFC-STYLE REQUIREMENTS (MANDATORY)

- MUST-COMPLIANCE RULE  
  All MUST requirements in this spec are mandatory. The generator MUST NOT
  silently downgrade a MUST requirement to a SHOULD or MAY.

- INFEASIBILITY OR CONFLICT HANDLING  
  If two or more MUST requirements conflict, or if any MUST requirement cannot
  be satisfied within the constraints of the requested implementation, the
  generator MUST:

  (a) explicitly identify each unsatisfied or conflicting MUST requirement,  
  (b) explain the minimal, concrete reason why each such MUST cannot be met, and  
  (c) propose one or more specific spec edits or design alternatives that would
      resolve the conflict or infeasibility.

- DEVIATION REGISTER (MANDATORY OUTPUT)  
  In the case of any unsatisfied or conflicting MUST requirement, the generated
  output MUST include a plain-text deviation register file. This file MUST:

  - list each unsatisfied or conflicting MUST requirement verbatim,
  - describe the minimal reason it could not be satisfied,
  - indicate whether the issue is due to an internal design conflict, an external
    constraint, or missing/ambiguous specification, and
  - reference the affected components or behaviours.

  The deviation register MUST be clearly named (e.g. DEVIATION_REGISTER.txt) and
  treated as part of the generated artefacts.

- PARTIAL OUTPUT RULE  
  The generator MAY still produce a partial implementation when MUST
  requirements cannot be met, but it MUST clearly label the implementation as
  non-compliant and MUST reference the deviation register as the authoritative
  record of outstanding violations.

- TRADEOFF PREFERENCE RULE  
  When multiple designs satisfy all MUST requirements, the generator SHOULD
  prefer designs that satisfy the greatest number of SHOULD requirements. Any
  significant SHOULD requirements that are not satisfied SHOULD be noted in
  comments or documentation, but need not appear in the deviation register.

- TESTING CLAIMS SAFETY RULE  
  If any MUST requirement in the testing or verification sections of the spec
  cannot be met, the generator MUST NOT claim that the test suite fully validates
  system behaviour. The generated documentation MUST explicitly state which
  behaviours, rules, or branches remain unvalidated and reference the deviation
  register.