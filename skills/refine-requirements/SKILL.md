---
name: refine-requirements
description: Use when requirements, specs, task descriptions, or feature requests are unclear, incomplete, ambiguous, or underspecified and the user wants concise refinements instead of a questionnaire.
---

# Refine Requirements

## Overview

Turn unclear or incomplete requirements into a concise list of user-owned refinements. Make reasonable default decisions, document them briefly, and expose uncertainty so the user can accept or modify the result.

This skill clarifies the shared contract between user or customer and implementer. It is not for documenting hidden implementation choices.

## Core Principle

Only document decisions the user should own.

A refinement belongs here if changing it would change what the user believes is being delivered.

A refinement does not belong here if it is merely one internal way to satisfy the same user-visible requirement.

## Include

Include refinements about:

- Goal or intent
- Scope boundaries, including exclusions
- Public behavior
- Function or API contract
- Inputs, outputs, mutability, side effects, and error behavior
- User-visible defaults
- Permissions, privacy, persistence, or data lifecycle
- Compatibility or integration expectations
- Acceptance-relevant constraints
- Ambiguous terminology
- Type of generated artifact (create a file or files vs. just content of a certain type)

## Exclude

Do not document:

- Obvious implications of the request
- Internal algorithms
- File layout, class structure, test strategy, or refactoring plans
- Framework or library choices unless user-visible
- Premature performance or scaling constraints
- Best-practices filler
- Options lists or questionnaires

## Output Format

Use this format:

```markdown
| Decision | Confidence | Rationale |
|---|---|---|
| ... | High / Medium / Low | ... |
```

Keep the list brief. Prefer 3-7 refinements. More than 10 usually means over-specification.

Each row should be understandable to a customer or user. Use direct requirement language and avoid implementation details.

Before output, delete any row that contradicts another row. A table must not contain two mutually exclusive decisions. If one row says the function returns a new slice, do not also include a row saying it mutates the input.

## Confidence

Use confidence to show which decisions would otherwise have been questions.

- **High**: strongly implied by the request or conventional for the domain.
- **Medium**: reasonable default, but another choice would also be plausible.
- **Low**: weakly implied; user should probably review.

Even low-confidence items should still be written as selected decisions, not questions.

Low confidence is for a chosen default whose basis is weak. Do not include rejected alternatives as low-confidence decisions. For mutually exclusive choices, choose one reasonable default and omit the other options unless they need to be explicitly excluded from scope.

Good:

> Primary metric: Monthly active users is treated as the dashboard's most important metric. Confidence: Low.

Bad:

> Which metric is most important?

Bad:

> Reverse the list in place (mutate the input). Confidence: Low.

This is bad when the table already selected non-mutating behavior. It is a rejected alternative, not a refinement.

## Rationale

Provide very brief rationale when it is not obvious or conventional. Leave the rationale cell blank when the decision speaks for itself.

## Style

Good:

> The function returns a new sorted slice and does not modify the input.

Bad:

> Consider whether to mutate the input or return a new slice.

Good:

> CSV export includes the currently visible filtered rows.

Bad:

> Use the existing table serializer to generate CSV.

Good:

> XLSX export is excluded.

Bad:

> XLSX can be added in a future iteration using a spreadsheet library.

## Example

Requirements:

Write a function that sorts a list of ints in Go.

Requirements refinements:

```markdown
| Decision | Confidence | Rationale |
|---|---|---|
| Output just the Go function definition and docs | High | Commentary, sample usage, etc. not implied by request |
| Function `SortInts` accepts and returns `[]int`. | Medium | Naming and signature are reasonable defaults for the request. |
| Return a new sorted slice without modifying the input. | Medium | Non-mutating behavior is safer unless mutation is requested. |
| Sort in ascending order. | High | Ascending is the conventional default. |
| Exclude custom comparators, descending order, generics, and explicit performance guarantees. | Medium | These are not implied by the request. |
```

## Common Mistakes

- Asking a questionnaire instead of making reviewable decisions
- Documenting internal implementation choices
- Adding best-practices filler that does not affect the delivered contract
- Turning every possible uncertainty into a row
- Listing mutually exclusive alternatives as separate decisions
- Writing options instead of decisions
- Using low confidence as an excuse to ask a question
