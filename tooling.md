# TOOLING SPECIFICATION (tooling.md)

## REPOSITORY ARTEFACTS (TOOLING PLANE, MUST)

The generated repository must include this specification verbatim in the root
directory. A README must be included and must do nothing except point to the
originating specification file.

## MAKEFILE OPERATIONS (TOOLING PLANE, SHOULD)

The repository should include a Makefile providing targets for deployment and
demonstrating example ingest and analysis requests. Targets must not require
interactive input.

## ASSUMPTIONS TRACKING MECHANISM (TOOLING PLANE, SHOULD)

The repository should include a plain-text assumptions log recording
implementation assumptions not explicitly stated in this specification. Each
assumption should reference relevant source files and searchable comment text.
