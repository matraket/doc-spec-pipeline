# Repository Guidelines

## How to Use This Guide

- Start here for cross-project norms.
- Each component has an `AGENTS.md` file with specific guidelines (e.g., `api/AGENTS.md`, `ui/AGENTS.md`).
- Component docs override this file when guidance conflicts.

## Rules

* Language:

  * Source code: English (EN).
  * Source comments: Spanish (ES).
  * Agent communication: Spanish (ES).
  * Context summarizations: always resume the conversation in Spanish (ES).

* File and folder names

  * File and folder names: snake_case or kebab-case
  * Allowed characters:
    * a-z
    * 0-9
    * Hyphen `-`
    * Underscore `_`
  * **CRITICAL**: Never use uppercase letters, spaces, or special characters.

* Coding and naming conventions

  * Classes: PascalCase (`MyClass`).
  * Variables/functions: camelCase (`myFunction`).
  * Constants: UPPER_CASE.

## Documentation Management

* Two skills manage the project specification:
  * `doc-spec-manager`: Navigates and queries the fragmented spec.
    Read its SKILL.md first for navigation instructions.
  * `doc-spec-generator`: Regenerates doc-spec-manager references
    from spec/ source files. Invoke after modifying spec/ documents.

* Documentation hierarchy:
  * `spec/` is the human-maintained source of truth.
  * `.agents/skills/doc-spec-manager/references/` contains
    agent-optimized fragments derived from spec/.
  * NEVER edit references/ directly. Always edit spec/ and regenerate.

* When implementing features:
  1. Use doc-spec-manager to find relevant RFs, USs, UCs.
  2. Follow traceability chain: RF -> RNF -> RNFT -> ADR -> BC -> US -> UC.

* File naming in references/:
  * All filenames are lowercase kebab-case (rnf-001.md, bc-identity.md).
  * Business codes inside files keep original format (RNF-001, BC-Identity).