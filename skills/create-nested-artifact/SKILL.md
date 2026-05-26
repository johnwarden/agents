---
name: create-nested-artifact
description: Use only when specifically asked to implement the given requirements.
---

# Create Nested Artifact

## Core Rule

Produce exactly one layer of artifact creation.

Choose one mode:

1. Produce the complete artifact directly.
2. Produce a parent artifact containing placeholders, plus requirement documents for the immediate subcomponents.

Do not recursively implement child subcomponents in the same invocation. The harness or caller is responsible for invoking the skill again for each child.

Output only artifact data. Do not include commentary, rationale, summaries, apologies, analysis, "here is" text, or descriptions of what changed unless the requested artifact itself requires those words.

## Inputs To Use

Use all provided context that affects the artifact:

- Requirements and acceptance criteria
- Design decisions, constraints, policies, style guidance, and learnings
- Existing artifact context when modifying or extending something
- Desired output mode: single file, multiple files, or raw non-file content

Make reasonable choices from the available context. Do not ask questions.

If requirements are impossible or contradictory, produce the smallest useful artifact and preserve the conflict as an explicit unresolved requirement placeholder inside the artifact, unless the input explicitly allows an error object.

## Output Modes

### File Artifacts

When producing one or more files, output valid JSON with exactly this envelope:

```json
{
  "artifacts": [
    {
      "filename": "path/to/file.ext",
      "content": "file contents"
    }
  ]
}
```

When file artifacts include subcomponents, add a `subcomponents` array:

```json
{
  "artifacts": [
    {
      "filename": "path/to/file.ext",
      "content": "file contents with subcomponent placeholders"
    }
  ],
  "subcomponents": [
    {
      "name": "subcomponent-name",
      "requirements": "# Requirements\n\n- Requirement details.\n"
    }
  ]
}
```

Rules:

- JSON must be valid.
- Do not use trailing commas.
- Do not include fields other than `artifacts`, `filename`, `content`, `subcomponents`, `name`, and `requirements` unless the input explicitly requires another schema.
- Represent each output file as one object in `artifacts`.

### Raw Content

When the requested artifact is raw non-file content, output only that raw content. Examples include a function definition, JSON object, paragraph, Markdown section, config object, or SQL query.

If raw content needs subcomponents, use valid JSON instead:

```json
{
  "content": "raw content with subcomponent placeholders",
  "subcomponents": [
    {
      "name": "subcomponent-name",
      "requirements": "# Requirements\n\n- Requirement details.\n"
    }
  ]
}
```

## When To Decompose

Create subcomponents when any of these are true:

- The complete artifact would exceed about 100 lines of code or structured content.
- The artifact has separately implementable parts already implied by the requirements, design, or acceptance criteria.
- A part is independently testable or independently replaceable.
- Implementing a part inline would make the parent artifact hard to inspect.
- The input explicitly asks for nested artifacts, decomposition, or subcomponents.

Prefer a complete artifact when the result is small, coherent, and easy to verify.

Do not create subcomponents merely to avoid making decisions. Do not create vague subcomponents. Do not create subcomponents for tiny helpers unless they are independently meaningful or already identified by the requirements.

## Placeholder Rules

Each placeholder must:

- Include the exact subcomponent name.
- Correspond to exactly one entry in `subcomponents`.
- Be easy for a later agent or script to find.
- Be valid or harmless in the surrounding artifact format whenever possible.
- Preserve enough surrounding structure that replacing the placeholder can produce a coherent final artifact.

Use the host artifact's natural comment or sentinel syntax:

```go
// subcomponent: reverse-ints-function
```

```python
# subcomponent: parse-input
```

```ts
// subcomponent: render-table
```

```md
<!-- subcomponent: introduction-section -->
```

```css
/* subcomponent: theme-variables */
```

```json
"__subcomponent:user-schema__"
```

```yaml
# subcomponent: deployment-settings
```

If no comment syntax is valid, use a string sentinel that is valid in the artifact syntax.

Do not use HTML comments in languages where they are invalid, such as Go, Python, JSON, YAML, JavaScript, or TypeScript.

## Subcomponent Names

Names must be:

- Unique within the output
- Stable and descriptive
- Lowercase when practical
- Suitable for filenames or folder names
- Hyphenated rather than spaced

Good names:

```text
reverse-ints-function
main-function
readme-usage-section
parser-tests
user-schema
```

Bad names:

```text
thing
part1
misc
handle stuff
subcomponent A
```

## Subcomponent Requirements

Each subcomponent's `requirements` value is the Markdown requirements document for that child.

Requirements documents must:

- Start with `# Requirements`.
- Be concise and local to the child.
- Include interface expectations directly in bullets when relevant.
- Include compatibility details needed by the parent artifact.
- Avoid inherited ancestor context unless directly necessary for local compatibility.
- Avoid separate `interface`, `rationale`, `why_subcomponent`, `learnings`, or `design_decisions` fields.

Example for code:

```md
# Requirements

- Define a Go function with signature `func reverseInts(xs []int) []int`.
- Return a new slice containing the input integers in reverse order.
- Do not mutate the input slice.
- Handle nil and empty slices without panic.
- The implementation must compile as part of package `main`.
```

Example for prose:

```md
# Requirements

- Write a Markdown section titled `## Usage`.
- Explain how to run the command.
- Include one shell command example.
- Keep the section under 150 words.
```

Example for JSON:

```md
# Requirements

- Produce a JSON object representing a user schema.
- Include `id`, `name`, and `email` fields.
- Mark all fields as required.
- Do not include comments or trailing commas.
```

## Multiple Files

Subcomponents may appear in any file. Usually each subcomponent belongs to one placeholder in one file.

If the same child content must be inserted in multiple places, make that explicit in the subcomponent requirements.

```json
{
  "artifacts": [
    {
      "filename": "main.go",
      "content": "package main\n\nimport \"fmt\"\n\n// subcomponent: main-function\n\n// subcomponent: reverse-ints-function\n"
    },
    {
      "filename": "README.md",
      "content": "# Reverse Integers\n\n<!-- subcomponent: usage-section -->\n"
    }
  ],
  "subcomponents": [
    {
      "name": "main-function",
      "requirements": "# Requirements\n\n- Define a Go `main` function.\n- Create a sample `[]int` with at least five integers.\n- Print the original slice.\n- Call `reverseInts` with the sample slice.\n- Print the reversed slice.\n- The function must compile as part of package `main`.\n"
    },
    {
      "name": "reverse-ints-function",
      "requirements": "# Requirements\n\n- Define a Go function with signature `func reverseInts(xs []int) []int`.\n- Return a new slice containing the input integers in reverse order.\n- Do not mutate the input slice.\n- Handle nil and empty slices without panic.\n- The function must compile as part of package `main`.\n"
    },
    {
      "name": "usage-section",
      "requirements": "# Requirements\n\n- Write a Markdown usage section.\n- Include the command `go run main.go`.\n- Briefly describe the expected output.\n- Keep the section under 100 words.\n"
    }
  ]
}
```

## Final Self-Check

Before outputting, verify:

- The response contains only artifact data.
- The chosen output mode matches the request.
- Any JSON envelope is valid JSON.
- There are no trailing commas or extra schema fields.
- Every placeholder has exactly one matching subcomponent.
- Every subcomponent has exactly one matching placeholder unless the requirements explicitly say otherwise.
- Placeholder syntax is valid or harmless for the host artifact.
- Subcomponent names are descriptive, unique, stable, and hyphenated.
- Every subcomponent requirements document starts with `# Requirements`.
- Requirements are local and include needed interface details.
- No subcomponent is recursively implemented in this invocation.
