# COMPLIANCE SPECIFICATION (Canonical)

## MUST: Compliance with RFC-style requirements

### MUST: MUST-compliance rule
All **MUST** requirements in this specification are mandatory.

The generator **MUST NOT** silently downgrade a **MUST** requirement to **SHOULD**
or **MAY**.

### MUST: Infeasibility or conflict handling
If two or more **MUST** requirements conflict, or if any **MUST** requirement
cannot be satisfied within the constraints of the requested implementation, the
generator **MUST**:

- explicitly identify each unsatisfied or conflicting **MUST** requirement,
- explain the minimal, concrete reason why each such **MUST** cannot be met, and
- propose one or more specific specification edits or design alternatives that
  would resolve the conflict or infeasibility.

### MUST: Deviation register
In the case of any unsatisfied or conflicting **MUST** requirement, the generated
output **MUST** include a plain-text deviation register file.

The deviation register **MUST**:
- list each unsatisfied or conflicting **MUST** requirement verbatim,
- describe the minimal reason it could not be satisfied,
- indicate whether the issue is due to an internal design conflict, an external
  constraint, or missing or ambiguous specification, and
- reference the affected components or behaviours.

The deviation register **MUST** be clearly named (for example,
`DEVIATION_REGISTER.txt`) and treated as part of the generated artefacts.

### MAY: Partial output rule
The generator **MAY** produce a partial implementation when **MUST** requirements
cannot be met.

In such cases, the generator **MUST**:
- clearly label the implementation as non-compliant, and
- reference the deviation register as the authoritative record of outstanding
  violations.

### SHOULD: Tradeoff preference rule
When multiple designs satisfy all **MUST** requirements, the generator **SHOULD**
prefer designs that satisfy the greatest number of **SHOULD** requirements.

Any significant **SHOULD** requirements that are not satisfied **SHOULD** be noted
in comments or documentation but **MUST NOT** appear in the deviation register.

### MUST: Testing claims safety rule
If any **MUST** requirement in the testing or verification sections of the
specification cannot be met, the generator **MUST NOT** claim that the test suite
fully validates system behaviour.

The generated documentation **MUST** explicitly state which behaviours, rules, or
branches remain unvalidated and **MUST** reference the deviation register.
