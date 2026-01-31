# Public Interface

## Purpose
This document defines which aspects of the system to be generated are, by definition the systems public interface.

## Scope
The scope of these definitions is across all the specs in this spec pack.

## Usage
There are several rules in the various specifications that make up this spec pack that define requirements
in terms of "the public interface", or sometimes "the public system interface". This document provides the
sole truth of truth for that definition.

- There MUST not be other definitions claiming to be the definition of the public interface specifically

## This project's public interface

This project's public interface is defined by the following set:

1. The http methods defined in system.md, including:
    -  their URLs
    -  their method
    -  their content types
    -  their CORS requirements
    -  the POST methods' payloads
    -  the POST methods' payload schema
    -  the POST methods' payload schema validation requirements
    -  the GET methods' return value schemas
    -  the GET methods' supported query methods (if any)
    -  the required set of HTTP status responses

- This list above of items that comprise the public interface is not the enumeration authority.
- It defines the classes of definitions in system.md that by definion enter the public interface.

2. Error handling

Every instance in this specification pack that defines an error condition to have an "error_id" is by definition part of the public interface.