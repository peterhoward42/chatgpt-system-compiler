# CLEANUP GO VARIANT (Server-side Phase)

## Scope

This document defines the mandatory server-side cleanup actions that apply when the
pack mandates the Go programming language.

This cleanup phase is executed after generation and before deliverables are finalized.

## Role

The role of this phase is to:
- normalize formatting of all Go source files, and
- enforce import hygiene rules that prevent unused or unclear dependencies from persisting in output.

This phase is permitted to modify generated artifacts. It MUST NOT introduce semantic changes
beyond those implied by the requirements in this document.

## Normative Language

The key words MUST, SHOULD, MAY, and related terms are to be interpreted as defined
in `compliance.md`. This document does not redefine their meaning.

## Definitions

For the purposes of this document:

- “Go file” means any file with the `.go` suffix in the deliverable workspace.
- “Selector qualifier” means the identifier `X` in a selector expression `X.Sel` in the Go AST
  (i.e., a `*ast.SelectorExpr` whose `X` is an `*ast.Ident`).
- “Import spec” means an `*ast.ImportSpec` within a file’s `import` declarations.
- “Blank import” means an import spec whose local name is `_`.

## Requirements

### R1. Formatting

The cleanup phase MUST run `gofmt` over all Go files in the deliverable workspace.

1. The `gofmt` invocation MUST be applied to every `.go` file, including generated code and tests.
2. The cleanup phase MUST fail only if `gofmt` itself fails to process a file.
3. If `gofmt` fails for a file, the failure MUST be recorded as a deviation in accordance with
   `compliance.md`. Whether processing continues is governed by the deviation handling rules
   defined in that file.

### R2. Import removal by AST qualifier usage

The cleanup phase MUST remove any import spec that is not referenced by a selector qualifier in the AST.

Normative interpretation:

1. For each Go file, the cleanup phase MUST parse the file into an AST using a Go parser that
   preserves comments.
2. The cleanup phase MUST compute the set `Q` of identifiers used as selector qualifiers in that file:
   - For every selector expression of the form `X.Sel`, if `X` is an identifier, add the identifier
     name to `Q`.
3. For each import spec in that file:
   - If the import has an explicit local name `name` (e.g., `foo "example.com/p"`), the import spec
     MUST be retained only if `name` is in `Q`.
   - If the import has no explicit local name (e.g., `"example.com/p"`), the import spec MUST be
     retained only if the default package name for that import path is in `Q`.
     - If the default package name cannot be determined reliably without type-checking, the cleanup
       phase SHOULD type-check the file or package to resolve the imported package name.
     - If resolution is not feasible, the cleanup phase MAY conservatively retain the import and
       MUST record a deviation explaining the uncertainty.
4. Any import spec that does not meet the retention rule above MUST be removed.

Dot imports (e.g., `. "example.com/p"`) do not create a qualifier identifier and therefore
MUST be removed under this rule.

### R3. Blank identifier imports with required side-effect comment

Blank identifier imports are permitted only when explicitly justified.

1. A blank import MUST be removed unless it includes a comment explaining the side effect
   being relied upon.
2. The explanatory comment MUST:
   - be attached to the import spec (either as a trailing comment on the same line or as an
     immediately adjacent comment in the import block), and
   - describe the intended side effect in concrete terms (for example: registering a driver,
     enabling a plugin, loading embedded assets).
3. If a blank import is retained, the cleanup phase MUST preserve the explanatory comment.

### R4. Idempotence

Running the cleanup phase multiple times SHOULD result in no further changes after
the first successful run.

## Reporting

If the cleanup phase cannot complete any MUST requirement for a specific file, it MUST
record a deviation in accordance with `compliance.md`, including:

- the file path,
- the failed requirement identifier (e.g., R1, R2, R3),
- a brief reason, and
- the action taken.

Unless `compliance.md` defines otherwise, deviations in this phase are non-blocking.