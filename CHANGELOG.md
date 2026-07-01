# Changelog

All notable changes to the TRAP prompt are documented here. The prompt follows
[Semantic Versioning](https://semver.org/): wording changes that meaningfully
alter audit behavior bump the version.

## [2.1.0] — 2026-06-30

Sharper injection and cryptography coverage. No breaking changes to the
methodology or output contract — every 2.0.0 verdict still holds.

### Added
- **Cryptography control (Phase 3).** Verifies algorithm *use*, not just presence: slow salted KDFs for passwords (not MD5/SHA), authenticated modes with unique nonces (not ECB/static IVs), constant-time secret comparison, and CSPRNG-sourced tokens.
- **Injection — trace it, don't spot it (Phase 3).** Requires tracing at least one concrete untrusted-input path from source to sink per injection class, mirroring the existing cross-tenant authorization trace. A library being "safe" is not a trace.

### Changed
- **Backend area** now calls out stack-specific dangerous sinks (`eval`, template injection, unsafe deserializers, ORM raw escapes, prototype pollution) and requires validation at *every* trust boundary — webhooks, queues, file contents, inter-service calls — not just the browser edge.
- **Remediation Loop** now requires keeping a passing exploit test as a committed regression test, so fixed vulnerabilities cannot silently return.

[2.1.0]: https://github.com/bikr/TRAP/releases/tag/v2.1.0

## [2.0.0] — 2026-06-26

First public release.

### Added
- **Rule 0 — Access Gate.** Refuses to audit code it cannot actually read, rather than hallucinating a report from a framework's typical structure.
- **Three-verdict Evidence Standard** — `VERIFIED` / `NOT PRESENT` / `UNKNOWN`, where "not present" is a finding and absence of evidence is never evidence of security.
- **Asserted ≠ Verified (false-confidence rule).** A declared-but-untested access control is reported as NOT PRESENT.
- **Coverage rule** and **Citation rule** to prevent sample-implies-full-coverage and locator-less "findings."
- **Severity × Confidence rubric** with an explicit ban on inflating severity to "play it safe."
- **11 phases:** Understand the System → Threat Model → Security Review → Privacy & Compliance → Failure Testing → Attack Mode → Performance & Scale → Code Quality → Out-of-Repo Operational Controls → Release Checklist → Automated Cross-Check.
- **Structured Output Format** (executive summary, findings table, exact code diffs, prioritized roadmap, verification appendix, scorecard, final verdict).
- **Mandatory Self-Challenge** pass and an iterative **Remediation Loop**.

[2.0.0]: https://github.com/bikr/TRAP/releases/tag/v2.0.0
