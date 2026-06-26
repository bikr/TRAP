---
name: TRAP missed something
about: TRAP failed to catch a real issue, or gave a false-confidence pass
title: "[MISS] "
labels: ["audit-gap"]
---

## What slipped through

<!-- Describe the issue TRAP should have caught. Redact anything sensitive. -->

## Where TRAP went wrong

- [ ] It marked something VERIFIED that wasn't actually proven
- [ ] It missed the control/finding entirely
- [ ] It softened a finding it should have held firm on
- [ ] Other:

## Minimal repro / pattern

<!-- A sanitized code snippet or description of the pattern that fooled it. -->

```text

```

## Suggested fix to the prompt

<!-- Which phase/rule needs tightening, and roughly how? -->

## Environment

- Tool used (Claude Code / Cursor / Copilot / web): 
- TRAP version: 
