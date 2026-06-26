# Contributing to TRAP

TRAP improves every time it's run against a real codebase and *misses* something,
or softens a finding it should have held firm on. Those misses are the most
valuable contributions you can make.

## Ways to contribute

- **Report a miss.** Open an issue describing what TRAP failed to catch, or where
  it produced a false-confidence ✅. Redact anything sensitive — a sanitized
  repro or description of the pattern is enough.
- **Tighten a phase.** Propose clearer or stronger wording for any of the 11 phases,
  the Evidence Standard, or the Output Format.
- **Add tooling coverage.** Phase 11 should know about the scanners people actually
  use. PRs that add a tool (with the right invocation) are welcome.
- **Improve docs.** README clarity, examples, and tool-compatibility notes.

## Guiding principles

Any change must keep TRAP's spine intact:

1. **Evidence over vibes.** Every claim carries a verdict and a locator.
   Never weaken the requirement to cite `file:line`.
2. **No false confidence.** Asserted ≠ verified. Untested controls are NOT PRESENT.
3. **Refuse rather than fake.** Rule 0 (Access Gate) is non-negotiable.
4. **The floor, not the ceiling.** Don't let the prompt imply it replaces a human
   security audit or a compliance assessment.

If your change would make TRAP *nicer* at the cost of being *honest*, it's the
wrong change.

## Pull request process

1. Edit the canonical prompt in [`TRAP.md`](TRAP.md).
2. If the change alters audit behavior, bump the version and add a versioned
   snapshot under [`prompts/`](prompts/) (e.g. `prompts/TRAP-v2.1.0.md`).
3. Record the change in [`CHANGELOG.md`](CHANGELOG.md).
4. Open the PR with a short note on *what audit behavior changes* and *why*.

## Versioning

- **Patch** (x.x.N): typo/clarity fixes that don't change audit behavior.
- **Minor** (x.N.0): new checks, phases, or tooling that extend coverage.
- **Major** (N.0.0): changes to the core methodology, verdict system, or output contract.

Thanks for helping keep the bar high. 🪤
