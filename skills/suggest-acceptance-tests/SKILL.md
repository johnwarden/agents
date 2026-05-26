---
name: suggest-acceptance-tests
description: Use when given requirements, specs, task descriptions, or subagent contracts and asked for concise suggested acceptance tests or a high-level verification brainstorm.
---

# Suggest Acceptance Tests

## Overview

Turn requirements into a short list of suggested acceptance tests. The goal is to identify what should be checked, not to design the full test procedure for the agent that will implement it.

## When to Use

Use this skill when the user wants:

- Suggested acceptance tests
- A quick verification brainstorm
- A compact list of what another agent should check

Do not use it when the user wants a full acceptance-criteria document, evidence matrix, or detailed test plan.

## Core Pattern

Read the requirements and produce a concise bullet list that covers:

1. The explicit requirements
2. Likely ways the output could look compliant while still being wrong
3. Important negative checks
4. Edge cases worth handing to the downstream test author

Keep each bullet high-level but concrete enough that another agent can turn it into a test.

## Quick Reference

Prefer bullets like:

- Check that ...
- Confirm that ...
- Manually review whether ...

Include:

- Mechanical checks when the requirement is precise
- Manual-review checks when judgment is genuinely required
- Negative checks when the artifact must avoid something

Avoid:

- Headings, matrices, tables, or long templates
- Pass/fail prose
- Verification methods or evidence sections
- Generic restatements such as "check that requirements are met"

## Implementation

Rules:

- Output only a short bullet list unless the user asks for more detail.
- Do not merely restate the requirements.
- Cover every important requirement at least once.
- Prefer specific observable checks over vague quality claims.
- Include negative or adversarial checks where superficial compliance is plausible.
- Use bounded manual review when the requirement cannot be checked mechanically.
- Leave detailed test design to the downstream agent.

Example style:

```markdown
- Check that the output contains no metric-unit tokens: g, kg, ml, L.
- Check that every nonblank line begins with a bullet marker.
- Check that the exact phrase or semantic equivalent of 1.2 lb boneless chicken thigh appears once.
- Manually review whether the ingredients plausibly support chicken tikka masala rather than a different curry.
- Confirm there are no headings, instructions, notes, or explanatory sentences outside the list.
```

## Common Mistakes

- Producing a full acceptance-criteria document instead of a compact brainstorm
- Writing implementation steps for the future test author
- Repeating the requirement text without identifying what to inspect
- Omitting negative checks because the happy path seems obvious
- Replacing concrete checks with vague judgments such as "good quality" or "correct output"
