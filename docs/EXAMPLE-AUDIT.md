# 🪤 TRAP Audit — QuoteFlow (Example Output)

> [!NOTE]
> **This is an illustrative example, not a real audit.** It shows the shape of a
> TRAP v2.0.0 run against a *fictional* app — **QuoteFlow**, a Next.js 14 +
> Supabase + Stripe + OpenAI B2B quoting tool — so you can see every section of
> the Output Format before running TRAP on your own code. File paths, line
> numbers, and findings are invented for demonstration.

**Rule 0 — Access Gate:** ✅ Repository access confirmed. Root: `~/projects/quoteflow` — 184 files across `app/`, `lib/`, `supabase/`, `components/`. Proceeding.

**Scope & Limits:** This audit is the floor, not the ceiling. It reliably surfaced OWASP-class issues, a secret exposure, and tenant-isolation gaps below. It did **not** evaluate the business-logic correctness of quote math, nor any runtime/load behavior. QuoteFlow stores its B2B customers' end-customer PII → **a human security audit + Data Processing Agreement review is required on top of this.**

---

## 1. Executive Summary

- **Overall Grade: D**
- **Production Recommendation: ❌ Do Not Deploy**
- The single most important reason: the Supabase **service-role key is bundled into the client** (`lib/supabase/client.ts:7`) and reachable in the browser network tab, which hands any visitor full read/write to every tenant's data. Treat it as already compromised and rotate before anything else.

## 2. Critical Issues (deployment blockers)

| # | Issue | One-line fix | Threat enabled |
| :-: | :-- | :-- | :-- |
| C1 | Service-role key shipped to client | Move to server-only env, rotate the key | Full cross-tenant DB read/write by anyone |
| C2 | `quotes` table has RLS *enabled but no policy proven by a cross-user test* | Add owner-scoped policy + cross-tenant test | IDOR across all tenants (Asserted ≠ Verified) |
| C3 | Stripe webhook has no signature verification (`app/api/webhooks/stripe/route.ts:12`) | Verify `stripe-signature` before trusting body | Forged "payment succeeded" → free plan upgrades |

## 3. Findings Table (excerpt — 9 of 23 shown)

| Severity | Confidence | Category | Issue | Verdict | Evidence | Fix |
| :-: | :-: | :-: | :-- | :-: | :-- | :-- |
| 🔴 Critical | Proven | Secrets | Service-role key in client bundle | NOT PRESENT (server/client split) | `lib/supabase/client.ts:7` | Server-only client; rotate key |
| 🔴 Critical | Proven | Payments | Webhook unsigned | NOT PRESENT | `app/api/webhooks/stripe/route.ts:12` | `stripe.webhooks.constructEvent()` |
| 🔴 Critical | Likely | Authorization | RLS enabled, no cross-tenant test | NOT PRESENT | `supabase/migrations/003_rls.sql:4` | Owner policy + proven test |
| 🟠 High | Proven | Rate limiting | No limit on `/api/ai/draft` (paid OpenAI call) | NOT PRESENT | `app/api/ai/draft/route.ts` | Per-workspace quota *before* dispatch |
| 🟠 High | Proven | Auth | JWT `exp` checked, `aud`/`iss` not | VERIFIED (partial) | `lib/auth/verify.ts:22` | Validate `aud` + `iss` |
| 🟡 Medium | Proven | API | Mass assignment on `PATCH /api/quotes/:id` | VERIFIED | `app/api/quotes/[id]/route.ts:31` | Allowlist updatable fields |
| 🟡 Medium | Likely | Frontend | `dangerouslySetInnerHTML` on quote notes | VERIFIED | `components/QuoteNote.tsx:18` | Sanitize (DOMPurify) |
| 🟢 Low | Proven | Infra | No HSTS header | NOT PRESENT | `next.config.js` (searched `headers`) | Add HSTS via `headers()` |
| ℹ️ Info | Proven | Code Quality | No test directory | NOT PRESENT | searched `**/*.test.*`, `__tests__/` | Add CI + tests |

> The full run lists all 23 findings. Note that **NOT PRESENT is a finding, not a pass** — C1, C2, and C3 are all "not present" controls, and they're the three that block the deploy.

## 4. Exact Code Changes (example for C3)

```diff
// app/api/webhooks/stripe/route.ts
- const event = await req.json();
+ const sig = req.headers.get('stripe-signature');
+ let event;
+ try {
+   event = stripe.webhooks.constructEvent(
+     await req.text(), sig, process.env.STRIPE_WEBHOOK_SECRET
+   );
+ } catch {
+   return new Response('Invalid signature', { status: 400 });
+ }
```

```diff
// lib/supabase/client.ts  — C1: stop shipping the service-role key to the browser
- export const supabase = createClient(
-   process.env.NEXT_PUBLIC_SUPABASE_URL!,
-   process.env.NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY!  // ⚠ reaches the bundle
- );
+ // Browser client uses the publishable anon key ONLY.
+ export const supabase = createClient(
+   process.env.NEXT_PUBLIC_SUPABASE_URL!,
+   process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
+ );
+ // Service-role client lives in a server-only module (see lib/supabase/admin.ts)
+ // and must never be imported from a Client Component.
```

## 5. Prioritized Remediation Roadmap

**Must do before real customer data**
- **C1** Rotate + move service-role key server-side — **S** (rotation logs nobody out; it's a data-plane key)
- **C2** RLS owner policy + cross-tenant integration test — **M** (depends on C1)
- **C3** Stripe webhook signature verification — **S**

**Should do before public launch**
- AI endpoint hard quota checked *before* dispatch — **M**
- HSTS + CSP headers — **S**
- JWT `aud`/`iss` validation — **S**

**Can do after launch**
- Sanitize quote-note HTML — **S**
- Add test suite + CI — **L**
- Fix N+1 on the quote-list query — **M**

## 6. Unknown / Operational Items

These cannot be proven from source and are **never** converted into a pass.

| Item | Why UNKNOWN | How to verify |
| :-- | :-- | :-- |
| OpenAI spend cap | Not in repo | OpenAI dashboard → Limits → set a hard cap + 50% alert |
| Backup restore-test | Not in repo | Supabase → Backups → run an actual restore drill |
| Service-role key rotation status | Exposed in C1 | Supabase → Settings → API → rotate; confirm the old key now 401s |
| Data region of storage bucket | Not in repo | Supabase → Storage → confirm region matches Phase 4 residency claim |

## 7. Verification Appendix

```bash
# C1 — confirm the secret reaches the browser bundle
grep -r "SERVICE_ROLE" app/ lib/ components/
npm run build && grep -r "service_role" .next/static/   # <-- a match here is the Critical

# C3 — confirm there is no signature check
grep -n "constructEvent\|stripe-signature" app/api/webhooks/stripe/route.ts   # <-- no match = unsigned

# Phase 11 — automated cross-check
npm audit --omit=dev
gitleaks detect --no-banner
```

## 8. Final Scorecard

| Security | Arch | Perf | Reliability | Maintainability | Privacy | DevOps | Prod-Readiness | Testing | Overall |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 2 | 6 | 5 | 4 | 6 | 3 | 4 | 2 | 1 | **3** |

## 9. Final Verdict

**No — I would not approve QuoteFlow for production at a company handling millions in annual revenue.**

Blockers, in order:
1. Rotate the exposed service-role key and move it server-side (C1).
2. Prove tenant isolation with a passing cross-tenant test, then enforce it with an RLS policy (C2).
3. Verify Stripe webhook signatures (C3).

Re-audit the delta once these land — a fix is only "done" when its specific exploit test passes.

---

> Want to see this on your own code? Copy [`../TRAP.md`](../TRAP.md) into your AI
> coding assistant and let Rule 0 fire.
