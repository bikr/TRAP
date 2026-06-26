<div align="center">

# 🪤 TRAP

### True Readiness Audit Prompt

**An adversarial, evidence-first production-readiness audit you run on your own codebase — before someone else runs it for you.**

[![Version](https://img.shields.io/badge/version-2.0.0-40DCA5?style=for-the-badge)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![Prompt](https://img.shields.io/badge/type-LLM%20prompt-8A2BE2?style=for-the-badge)](TRAP.md)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-success?style=for-the-badge)](CONTRIBUTING.md)

[![Works with Claude](https://img.shields.io/badge/Claude-D97757?style=flat-square&logo=anthropic&logoColor=white)](#-which-tools-does-it-work-with)
[![Works with Cursor](https://img.shields.io/badge/Cursor-000000?style=flat-square&logo=cursor&logoColor=white)](#-which-tools-does-it-work-with)
[![Works with Copilot](https://img.shields.io/badge/GitHub%20Copilot-181717?style=flat-square&logo=github&logoColor=white)](#-which-tools-does-it-work-with)
[![Works with ChatGPT](https://img.shields.io/badge/ChatGPT-412991?style=flat-square&logo=openai&logoColor=white)](#-which-tools-does-it-work-with)

[**📋 Copy the prompt**](TRAP.md) · [**🚀 Quick start**](#-quick-start) · [**🧭 The 11 phases**](#-how-it-works-the-11-phases) · [**☕ Support**](#-support)

</div>

---

> [!WARNING]
> **"It works on my machine" is not a security posture.** Vibe-coded apps ship with exposed service keys, unenforced tenant isolation, and rate limits that exist only in the prompt that asked for them. TRAP is the cold-shower review that finds those things *before* your users — or an attacker — do.

## What is TRAP?

TRAP is a single, copy-pasteable prompt that reprograms your AI coding assistant into a **10-person adversarial review board** — Staff Engineer, Principal Security Engineer, Pentester, OWASP Specialist, DBA, Cloud Architect, Privacy & Compliance Consultant, Release Manager, and more — speaking in one consolidated voice.

Its only job: decide whether your app is **genuinely production-ready**, and hand you the exact path to make it so.

It is built around one uncomfortable idea:

> **An invented finding is worse than a missing one, because it manufactures false confidence.**

So TRAP refuses to guess. Every single claim it makes must carry a verdict:

| Verdict | Meaning |
| :-- | :-- |
| ✅ **VERIFIED** | "I read the code/config that proves this." Cites `file:line`, route, middleware, or policy. |
| ❌ **NOT PRESENT** | "I searched for this control and it does not exist." States the search terms so you can reproduce it. **Not present is a finding, never a pass.** |
| ❓ **UNKNOWN** | "Can't prove this from source." Tells you exactly which console/command/file *would* prove it. |

If it can't point at the code, it doesn't get to call you secure.

## Why it's different

Most "review my app" prompts are flattery machines — they pattern-match a framework's typical structure and hand back a reassuring ✅. TRAP is engineered specifically to **resist** that.

- 🚫 **Access Gate (Rule 0).** No repo access? It refuses to audit and tells you so — instead of hallucinating a report from vibes.
- 🔬 **Asserted ≠ Verified.** An access-control policy that *exists* but isn't proven by a real cross-tenant test is reported as **NOT PRESENT**. Enabling a guard you never tested is treated as *worse* than not having one.
- 🎯 **Coverage rule.** Finite sets (routes, env vars, migrations, upload handlers) get enumerated and audited individually — no implying full coverage from a sample.
- ⚖️ **Severity × Confidence on every finding.** And it's forbidden from inflating severity on unverified hunches to "play it safe."
- 🗡️ **Attack Mode (Phase 6).** It stops reviewing and starts attacking — likelihood, impact, difficulty, *and whether your logging would even catch it.*
- 🪞 **Mandatory Self-Challenge.** Before it's done, it audits *its own report* for over-claiming and downgrades anything it inferred rather than read.
- 🔁 **Remediation Loop.** A fix is only "done" when its specific exploit test passes — not when the code merely *looks* correct.

> [!IMPORTANT]
> **TRAP is the floor, not the ceiling.** It reliably catches OWASP-class issues, secret exposure, IDOR/tenant gaps, missing rate limits, and config holes. It does **not** reliably catch business-logic flaws, subtle race conditions, or anything needing live runtime behavior. For regulated data (HIPAA/PCI/COPPA), it explicitly tells you a human audit is still required. A clean TRAP report is a strong signal — not a compliance certificate.

## 🚀 Quick start

1. **Open** your AI coding assistant in your project (Claude Code, Cursor, Copilot Chat, etc.) so it can actually read your files.
2. **Copy** the full prompt from **[`TRAP.md`](TRAP.md)** — everything below the divider.
3. **Paste** it as a single message.
4. **Watch Rule 0 fire.** It first confirms it can see your code (root path + rough file count). If it can't, it stops. Good — that's the point.
5. **Read top-down.** The Executive Summary gives you a letter grade and a `✅ Ready / ⚠️ Ready with Conditions / ❌ Do Not Deploy` call. The Remediation Roadmap gives you the ordered path out.
6. **Fix, then loop.** Hand back your changed files and ask it to re-audit only the delta. Repeat until `✅ Ready` with zero open Critical/High findings.

```text
┌────────────────────────────────────────────────────────────┐
│  Give AI repo access  →  Paste TRAP.md  →  Rule 0 gate      │
│        ↓                                                    │
│  11 phases of evidence-based audit                          │
│        ↓                                                    │
│  Graded report + exact diffs + remediation roadmap          │
│        ↓                                                    │
│  Apply fixes  →  Re-audit the delta  →  ✅ Ready            │
└────────────────────────────────────────────────────────────┘
```

> 💡 **Tip:** Run TRAP against an assistant that has genuine, full read access to the repository. The audit is only as good as the code it can actually see — that's why Rule 0 exists.

## 🧭 How it works: the 11 phases

TRAP forces a build-then-break sequence. It won't review controls until it has modeled the system, and it won't stop at the happy path.

| # | Phase | What it does |
| :-: | :-- | :-- |
| **1** | 🗺️ **Understand the System** | Maps every component, provider, trust boundary, and **data residency** region — with a text architecture diagram. No review until the model is grounded in files. |
| **2** | 🎯 **Threat Model** | Names *this app's* critical assets, threat actors, and attack surfaces. Every later finding ties back to a threat here. |
| **3** | 🛡️ **Security Review** | OWASP Top 10 + modern SaaS: auth, authz/IDOR, DB/RLS, API, frontend XSS, secrets split, uploads, **AI security**, payments, rate limiting, bot protection, infra, DevOps. |
| **4** | ⚖️ **Privacy & Compliance** | GDPR/CCPA, data inventory + residency, DPAs for processor PII, retention/erasure — all flagged for counsel, never asserted as "compliant." |
| **5** | 💥 **Failure Testing** | The unhappy paths: tampered JWTs, lockouts, enumeration leaks, path traversal, race conditions, replay. Maps each classic test to its magic-link/OAuth equivalent. |
| **6** | 🗡️ **Attack Mode** | Becomes the adversary. Every attack scored on Likelihood · Impact · Difficulty · **Detection** · Mitigation. |
| **7** | ⚡ **Performance & Scale** | N+1s, indexes, caching, pooling, bundle size — separating "fails at launch load" from "fails at growth." |
| **8** | 🧹 **Code Quality** | Architecture, tech debt, **test coverage** (NOT PRESENT if none), error handling, config hygiene. |
| **9** | 🔌 **Out-of-Repo Controls** | The things source can't prove: paid-API hard caps, billing alerts, **secret rotation**, backup restore-tests, prod monitoring. ✅ CONFIRMED or ❓ UNKNOWN — never hand-waved. |
| **10** | 📋 **Release Checklist** | 30+ go-live items, each ✅ / ❌ / ❓ with evidence. |
| **11** | 🤖 **Automated Cross-Check** | Reconciles `npm audit` / `gitleaks` / CodeQL against the manual pass — and resolves every disagreement. |

Then comes the **Output Format** (executive summary → findings table → exact code diffs → prioritized roadmap → scorecard → final verdict), the **Self-Challenge**, and the **Remediation Loop**.

<details>
<summary><strong>📄 See the final Output Format TRAP produces</strong></summary>

1. **Executive Summary** — letter grade + deploy recommendation + the single most important reason.
2. **Critical Issues** — every blocker first, with its one-line fix and the threat it enables.
3. **Findings Table** — Severity · Confidence · Category · Issue · Verdict · Evidence (`file:line`) · Fix.
4. **Exact Code Changes** — real diffs matching your stack and conventions.
5. **Prioritized Remediation Roadmap** — *Must do before real customer data* · *Should do before public launch* · *Can do after launch*, each with S/M/L effort.
6. **Unknown / Operational Items** — everything unverifiable, with how to verify it. Never converted into a pass.
7. **Verification Appendix** — the exact commands/greps/queries so another engineer can independently reproduce the result.
8. **Final Scorecard** — 1–10 across Security, Architecture, Performance, Reliability, Maintainability, Privacy, DevOps, Production Readiness, Testing, Overall.
9. **Final Verdict** — *"Would you personally approve this for production at a company handling millions in annual revenue?"*

</details>

## 🧰 Which tools does it work with?

TRAP is plain text — it runs anywhere you can paste a prompt and give the model repo access. It shines most where the assistant has **genuine file access**:

| Tool | Repo access | Notes |
| :-- | :-: | :-- |
| **Claude Code** | ✅ Full | Ideal — true filesystem access, runs `npm audit`/`gitleaks` for Phase 11. |
| **Cursor** | ✅ Full | Point it at the workspace; great for the per-file citation rule. |
| **GitHub Copilot Chat** | ✅ Workspace | Use `@workspace` so it reads real files. |
| **Windsurf / Cline / Aider** | ✅ Full | Any agentic IDE with repo access works well. |
| **ChatGPT / Claude.ai (web)** | ⚠️ Paste-in | Works, but you must paste the file tree + contents. Rule 0 will demand it. |

> [!NOTE]
> If the assistant *can't* read your actual source, TRAP is designed to **refuse** rather than fake it. That refusal is a feature, not a bug.

## 🗂️ Repository layout

```
TRAP/
├── README.md              ← you are here
├── TRAP.md                ← the canonical prompt (latest)
├── prompts/
│   └── TRAP-v2.0.0.md     ← versioned snapshots
├── CHANGELOG.md           ← what changed between versions
├── CONTRIBUTING.md        ← how to propose improvements
├── LICENSE                ← MIT
└── .github/               ← issue templates, funding
```

## 🤝 Contributing

TRAP gets sharper every time it's run against a real codebase and *misses* something. If you find a gap, a false-confidence trap it didn't catch, or a phase that needs tightening:

- Open an [issue](../../issues) describing what slipped through (redact anything sensitive).
- Or send a PR — see **[CONTRIBUTING.md](CONTRIBUTING.md)**.

The prompt is versioned with [semantic versioning](https://semver.org/). Wording changes that alter audit behavior bump the version.

## ☕ Support

If TRAP caught something before it cost you a 2 a.m. incident, a leaked key rotation, or a breach disclosure — consider buying me a coffee. It directly funds keeping this prompt sharp.

<div align="center">

<a href="https://www.buymeacoffee.com/bikra" target="_blank">
  <img src="https://img.shields.io/badge/Buy%20me%20a%20coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me A Coffee" />
</a>

<br/>

<a href="https://www.buymeacoffee.com/bikra" target="_blank">
  <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" height="50" width="210" />
</a>

</div>

## 📜 License

[MIT](LICENSE) © [bikr](https://github.com/bikr). Use it, fork it, adapt it for your team. Attribution appreciated, not required.

---

<div align="center">

**Don't ship hope. Ship evidence.**

⭐ If TRAP saved you a bad deploy, star the repo so the next person finds it.

</div>
