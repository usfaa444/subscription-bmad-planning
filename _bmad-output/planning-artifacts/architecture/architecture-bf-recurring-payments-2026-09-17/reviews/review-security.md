# Security / compliance review

**Artifact:** `ARCHITECTURE-SPINE.md` (bf-recurring-payments, 2026-09-17)  
**Lens:** Ad-hoc security / compliance — Burkina Faso / WAEMU receivables  
**Binds checked:** secrets, sandbox isolation, MSISDN/PII, webhook HMAC, API keys, no e-money / Customer wallets, reminder-then-pay vs auto-debit, KYC gate, consent audit  
**Sources:** spine ADs 1–14 + conventions; PRD FR-4/5/11/12/16/17/20/24/31/38/43/48/50, NFR-6/7/10/12, §12–13, §17; addendum regulatory notes  
**Date:** 2026-09-17

## Verdict

**Accept-with-gaps — product-compliance locks are adopted; the operational security envelope is not closed.**

Reminder-then-pay, vacant `DirectDebitPort`, no Customer-visible wallet, pass-through-until-counsel, KYC as a payout-only gate, hashed live API keys shown once, `SandboxRail` + no live Orange credentials in the sandbox env, HMAC-SHA256 outbound webhook shape, and consented-text versioning would stop the main WAEMU / BCEAO / copy-law divergences **if implementers follow the Rules as written**.

They will not. The spine leaves inbound Rail callback authentication, T+1 fund custody, HMAC replay window and signing-secret rotation, Staff PIN hashing, tenant/`livemode` query isolation, PII in queues/traces, KYC document custody, and Sandbox→`KycGate` leakage as inventable. Two teams would ship different launch-critical controls. This is not a license to reopen AD-4 / AD-7 / AD-10 / AD-11 product posture — it is a demand to bind the missing mechanics.

## What the spine already locks (do not reopen)

These Rules are enforceable and match PRD §5 / §13. Findings below do not weaken them.

| Lock | Where | Why it is enough as posture |
| --- | --- | --- |
| Collect only after Customer confirm on **that** `CollectionAttempt`, or Cash Mark-paid | AD-4 | Matches FR-20. Jobs may SMS; they must not debit. |
| `DirectDebitPort` vacant, NotImplemented, no MVP call, no Sandbox debit toggle | AD-2, AD-4 | Matches FR-48. Later enablement needs a **new** `StandingAuthorization` text version (AD-4 / AD-11). |
| Copy must not claim prélèvement automatique / auto-debit | AD-4 | Matches FR-11 copy law. |
| No Customer-visible wallet; SOLDE must not show a platform-held balance | AD-7, AD-13 | Matches FR-38, FR-30, §5. |
| Do not implement platform float until counsel writes the control | AD-7, Deferred | Matches §8.1 / license guardrail. |
| `TakeRatePolicy` 0% on Sandbox / cash / fail; first-success is live MoMo only | AD-10, AD-11 | Matches FR-40–42, FR-50. |
| `KycGate` blocks **payouts** not invoicing; sum = live MoMo + Cash Mark-paid; 500 000 XOF / CountryPack month or payout-destination change | AD-10 | Matches FR-43. Informal start stays FR-44. |
| API keys hashed at rest; live secret shown once; Hosted Checkout Session API accepts no Rail PIN/OTP/secrets | AD-11 | Matches NFR-6, FR-17. |
| Sandbox: separate key namespace + `SandboxRail`; no live Orange credentials in sandbox env | AD-11, AD-14 | Matches FR-5, FR-50. |
| Prod logs mask MSISDN to last 4; consent bodies not in debug logs | AD-11, AD-14 | Partial NFR-7. |
| Store consented text version on `StandingAuthorization` | AD-11 | Partial NFR-12 / FR-11. |
| Outbound webhooks: HMAC-SHA256 over `timestamp.rawBody`; `X-Webhook-Signature` `t=` + `v1=`; at-least-once; debit at-most-once per Attempt token; replay must not re-debit | AD-8 | Partial NFR-10 / FR-16 / FR-24. |
| Environments `local` \| `sandbox` \| `production`; diagram shows isolated sandbox Postgres | AD-14 | Directionally correct isolation. |
| Customer: Magic Link or OTP only, TTL 15 minutes; no Customer passwords | AD-3, AD-11 | Matches FR-12. |
| Staff PIN when enabled is required for Cash Mark-paid, retry, payout change, API-key reveal, Cancel, Client Record remove, Write-off | AD-11 | Matches FR-4 privileged set (plus Cancel / remove / Write-off). |

## Findings

### Critical

None that break the adopted posture itself. The highest items are **high** because they let an implementer violate that posture without contradicting a written Rule.

### High

- **[high] Inbound Rail callbacks and `rail.poll` are unsigned and semantically unbound (AD-2, AD-4, AD-8, AD-13)** — AD-8 specifies only **outbound** Merchant webhooks. The live Orange path needs an inbound status (callback and/or `rail.poll`). Neither port contract nor Rule says: (1) inbound Rail callbacks are authenticated (adapter-verified signature / mTLS / shared secret) before any aggregate mutation; (2) `rail.poll` is **status-only** and must not call `PaymentRailPort.collect`; (3) a forged or replayed “success” cannot mark `collected`, mint Take-rate, or advance Payout Clock. BullMQ is at-least-once. *Guard:* Add to AD-8 (or a new secrets/callbacks AD): inbound Rail messages authenticate at the adapter; domain applies status only to an existing Attempt token that already has Customer confirm (or Cash Mark-paid); `rail.poll` never collects; fail closed on auth miss. *If it ships as-is:* a forged Orange callback or a worker that “helpfully” re-collects on poll becomes silent auto-debit or false settlement — the exact FR-20 / SM-4 failures the spine claims to prevent.

- **[high] T+1 pass-through does not bind who holds XOF overnight (AD-7, Deferred legal float)** — Customer-paid is immediate; Merchant-received is T+1 CountryPack business day. Someone holds value in between. AD-7 forbids a Customer-visible wallet and says “prefer pass-through” / “do not implement platform float until counsel writes the control,” but it does not forbid a platform-controlled Orange MSISDN / aggregator sub-account used as overnight park. WAEMU/BCEAO e-money and payment-institution risk is **custody**, not the SOLDE label. *Guard:* Rule: T+1 delay, if any, lives at the licensed Rail / aggregator counterparty. The platform must not be the legal holder of Customer-paid funds. A platform-controlled settlement account is float and is Deferred until counsel writes the control. Ledger `settled` means the licensed party confirmed credit to Payout Destination — not that we released a house balance. *If it ships as-is:* first live volume looks like an EME / payment-institution activity; §5 and §13 become marketing.

- **[high] Sandbox `collected` is excluded from Take-rate but not from `KycGate`; tenant / livemode isolation is a diagram, not a query rule (AD-6, AD-10, AD-11, AD-14)** — AD-11: Sandbox `collected` does not increment first-success or Take-rate. AD-10 `KycGate` sums “live MoMo collected XOF + Cash Mark-paid.” If SandboxRail writes the same `LedgerEntry` / `collected` shape on the same `merchant_id` (Stripe-style `livemode` in one DB) or if sandbox/prod credentials share Redis/queues, fake collects can trip or dilute the 500 000 XOF payout gate, and jobs can leak across envs. AD-14 draws a separate sandbox Postgres but never says: production process refuses sandbox keys **and** has no `SandboxRail` collect path; sandbox process has zero live Orange credentials (already said) **and** cannot resolve production Merchant payout credentials; every tenant read/write is `merchant_id` (+ `livemode` if shared DB) or it is a defect. *Guard:* Bind fail-closed env isolation (config lint: live Orange vars forbidden when `APP_ENV=sandbox`; sandbox key prefix rejected in production). If one DB: `livemode` on money/consent rows; `KycGate` and `TakeRatePolicy` read live-only. If two DBs: no shared Redis/BullMQ. Mandatory `merchant_id` predicate on every tenant query (tests that fail without it). *If it ships as-is:* informal KYC gate is bypassable or false-positive; cross-tenant Session/Obligation reads; sandbox job hits live Orange.

- **[high] PII / KYC minimization is logs-only; queues, traces, and ID documents are unbound (AD-11, AD-14, NFR-7, NFR-12, §17)** — Prod logs mask MSISDN to last 4; consent bodies stay out of debug logs. That does not cover BullMQ/Redis payloads (`sms.send`, `reminder.dispatch`, `webhook.deliver`), OpenTelemetry attributes, Next.js BFF logs, exception trackers, or inbound SMS vendor webhooks. PRD classifies MSISDN, consent text, and ledger as sensitive, and FR-43 uploads NINEA/RCCM/national ID. No AD for: field policy on job payloads; trace allowlist; KYC image/PDF encryption and operator ACL; retention (life of Merchant relationship + documented minimum); French privacy notice as a shipped surface (NFR-7). Fly.io `ams` stores BF Customer/Merchant PII in the EU; PRD §17 residency is “if legal allows — revisit with counsel.” Spine adopts `ams` without a go-live counsel gate. *Guard:* Mask or tokenize MSISDN in all prod telemetry and job payloads (resolver in `SmsPort` / rail adapter only). KYC documents encrypted, operator-only, not in debug logs or Merchant-export zips beyond the uploader. Retention + no-sale already in PRD — restated as an AD. Production PII in `ams` requires a recorded counsel accept (or a later in-region AD). *If it ships as-is:* full MSISDN and ID scans in Redis/OTel; BCEAO-adjacent KYC dossier with no custody; residency surprise at first incident.

### Medium

- **[medium] HMAC shape is specified; verification policy and FR-16 rotation are not (AD-8, NFR-10, FR-16)** — `HMAC-SHA256(timestamp + "." + rawBody)` and `X-Webhook-Signature: t=, v1=` are the right Stripe-shaped contract. Missing: receiver skew window (e.g. reject `|now - t| > 5m`); constant-time compare; signing secret ≠ API key; secret hashed-or-encrypted at rest with **recoverable** ciphertext (Merchants cannot verify if you only store a hash of the webhook secret); rotation from Developers with dual-`v1` overlap; revoke-on-rotate. *Guard:* Add those four sentences to AD-8. *If it ships as-is:* replayed callbacks forever; rotation ships as “create a new API key”; leaked webhook secret lives forever.

- **[medium] NFR-6 assigned storage mechanics to architecture; Staff PIN hash and API-key algorithm/revocation are missing (AD-11)** — Live keys hashed + shown once is necessary but not sufficient. Unspecified: KDF (Argon2id / scrypt, not raw SHA-256); key prefix namespace (`sk_live_` / `sk_test_` or equivalent) enforced at the HTTP adapter; revoke and rotate without a second undocumented path; look up by key id prefix then verify hash (do not scan all rows). Staff PIN “cannot be recovered in plaintext” (NFR-6) is not in AD-11 — only the privileged-action list. *Guard:* Staff PIN hashed at rest, never logged, never recovered; API keys Argon2id/scrypt + prefix + revoke. *If it ships as-is:* plaintext PIN in Merchant row; SHA-256 keys rainbow-tabled; stolen key lives until a manual SQL delete.

- **[medium] Consent audit stores a text version, not the NFR-12 trail (AD-11, AD-9, NFR-12, FR-11, FR-31)** — NFR-12: Standing Authorization text version, **timestamps**, Collection Attempt **tokens**, **Cash Mark-paid actor**, **payout-destination changes**. AD-11 only names the text version. Cash Mark-paid actor and payout-destination history have no aggregate/outbox requirement. Retention and append-only / no silent rewrite of consented text are unset. FR-11 / FR-31 shared-phone and STOP≠ANNULER disclosures must be inside the versioned body or the audit cannot prove Instruction 001-01-2024-style consent. *Guard:* `StandingAuthorization` stores `{text_version_id, text_hash, locale, accepted_at}`; `CollectionAttempt` token immutable; Cash Mark-paid writes `actor_staff_id`; Payout Destination changes are an append-only event; consented body includes reminder-then-pay, per-attempt confirm, PAUSE/REPRENDRE/ANNULER, and STOP≠Cancel. *If it ships as-is:* dispute and regulator ask “what did Fatou agree to, who marked cash, who moved payout?” and the ledger cannot answer.

- **[medium] Hosted Checkout / Magic Link / OTP abuse and shared-phone token leakage are TTL-only (AD-3, AD-11, FR-12)** — 15-minute TTL and “replay after expiry fails” are assumed. Missing: OTP rate limit and lockout per MSISDN; Magic Link single-use; link tokens not echoed in server logs or `Referer` to third parties; checkout Session URLs unguessable (ULID with CSPRNG randomness); bind confirm to the identified Customer of that Session. Shared-phone disclosure is a PRD consent line, not a spine Rule. *Guard:* Rate-limit OTP; single-use Magic Link; no token in OTel; Session id entropy; confirm requires prior identify on that Session. *If it ships as-is:* SMS-bomb; harvested Magic Links on a shared Android pay Fatou’s Plan.

- **[medium] Webhook delivery is an outbound SSRF and confused-deputy surface (AD-8)** — Platform POSTs to Merchant-controlled URLs from workers that also hold Rail/Postgres credentials. No block of link-local / RFC1918 / metadata IPs; no scheme allowlist (`https` only in prod); no redirect follow policy. *Guard:* HTTPS-only in sandbox/production; deny private/link-local/metadata; do not follow redirects to those ranges; timeout + max body. *If it ships as-is:* a Merchant webhook URL becomes an internal port scanner from Fly `ams`.

- **[medium] Portal BFF and public API share one Nest process (AD-1, AD-11, AD-12)** — One deployable API + workers is the right monolith call. Authn modes collide: Merchant session + Staff PIN vs `Authorization: Bearer` hashed API key vs Customer Magic Link/OTP. Missing: separate authn guards per surface; CSRF on cookie portal mutations; API key cannot call Staff-PIN portal actions (Cash Mark-paid, Write-off, payout change) unless a later AD says so (PRD privileged actions are portal/Staff PIN). *Guard:* Explicit guard matrix. Cookie session ≠ API key. Privileged money actions stay Staff-PIN portal (or equivalent step-up), not raw Bearer. *If it ships as-is:* leaked sandbox key marks cash and moves payout.

- **[medium] Idempotency-Key is not scoped (AD-8, conventions)** — Session create and Attempt confirm require `Idempotency-Key`. Unspecified: scope = `merchant_id` + route + key; TTL; payload hash mismatch returns a conflict, not a second debit. *Guard:* Scope keys to merchant + route; store request hash. *If it ships as-is:* cross-merchant key collision or replay with a mutated body.

### Low

- **[low] Forbidden auto-debit strings have no mechanical check (AD-4)** — Rule is copy-only. CountryPack / i18n CI should fail the build on “prélèvement automatique”, “auto-debit”, “set and forget”, “until automatic debit is enabled.” Cheap, high leverage.

- **[low] Operator visibility is granted without an operator identity (AD-14, NFR-4)** — “Operators see Adapter-level Rail outcome without exposing secrets to Merchants” needs an operator role, audit log, and no raw Rail credentials in that UI.

- **[low] TLS, HSTS, cookie flags, and encryption-at-rest are assumed, not adopted** — Fly Managed Postgres disk encryption is likely; say so. Portal cookies: `Secure`, `HttpOnly`, `SameSite=Lax` (or `Strict` where it does not break checkout). Terminate TLS at the edge. Not exotic — but this spine is the place that prevents a Compose-with-plaintext-HTTP story from reaching sandbox.

- **[low] ARCEP / STOP is product-complete in the PRD and only verb-listed in the spine (AD-13)** — Architecture does not need to rewrite FR-31. It should say SMS adapter and `reminder.dispatch` honor the STOP allowlist so a worker cannot invent marketing SMS. Transactional-only MVP makes this a footgun, not a current violation.

- **[low] `refunded` reserved / no Merchant-initiated refund (AD-7)** — Correct non-goal. Ensure Rail Reversal (FR-56) cannot be implemented as a platform-originated payout to the Customer wallet (that is sending e-money). Reversal is a Rail-originated status.

## Lens walkthrough

### Secrets

Covered: no Rail PIN on Merchant APIs (AD-11); live Orange secrets forbidden in sandbox (AD-11, AD-14); operators must not expose secrets to Merchants (AD-14).  
Gap: inbound Rail secret; webhook signing-secret storage/rotation; Staff PIN hash; API-key KDF/revoke; secret scanning / env lint as an AD; ClockPort does not protect leaked OTP if logs print it.

### Sandbox isolation

Covered: separate key namespace, `SandboxRail`, no live Orange creds, separate env in the diagram, Sandbox `collected` ↛ Take-rate.  
Gap: KycGate; shared Redis/queues; production accepting sandbox keys; compose pointing at live Orange; one-DB `livemode` unspecified.

### MSISDN / PII

Covered: mask last 4 in prod logs; no Customer email harvest is a PRD NFR the spine does not contradict; CountryPack parse.  
Gap: jobs, traces, SMS vendor retention, KYC documents, residency counsel gate, tenant filters.

### Webhook HMAC

Covered: algorithm, header format, at-least-once, retry schedule, dead-letter, replay ≠ re-debit.  
Gap: skew window, constant-time compare, rotation (FR-16), secret ≠ API key, inbound Rail auth, SSRF.

### API keys

Covered: hashed at rest; live shown once; Bearer on `/v1`; sandbox vs live namespace.  
Gap: KDF, prefix enforcement, revoke/rotate, privilege boundary vs portal Staff PIN.

### No e-money / Customer wallets

Covered: no Customer-visible wallet; no SOLDE house balance; float Deferred.  
Gap: T+1 custody party; reversal-as-payout; platform settlement account.

### Reminder-then-pay vs auto-debit

Covered: AD-4 is the strongest AD in the spine; vacant port; new text version later; copy ban.  
Gap: `rail.poll` / inbound callback can reintroduce auto-debit without editing AD-4; no CI on forbidden strings.

### KYC gate

Covered: payout-only; 500 000 XOF live MoMo + cash; destination change; invoicing continues; accepted flag / manual review.  
Gap: document custody; who may flip `accepted`; in-flight payout race; Sandbox sum; payout MSISDN ownership proof (name/MSISDN match) — product risk, not EME KYC (PRD is explicit the gate is **not** EME compliance).

### Consent audit

Covered: text version on `StandingAuthorization`; outbox in the same TX as the aggregate (AD-8) is a good audit substrate.  
Gap: NFR-12 field list, immutability, STOP/shared-phone in the versioned body, retention.

## Recommended AD deltas (autofix candidates)

For Finalize — do not reopen posture; add mechanics:

1. **AD-8** — Inbound Rail auth; `rail.poll` status-only; HMAC skew ≤ 5 minutes; constant-time compare; signing secret rotatable (FR-16), stored recoverable-encrypted, distinct from API keys; dual-key overlap on rotate; webhook SSRF deny-list.
2. **AD-7** — T+1 custody stays at licensed Rail/aggregator; platform-controlled park = Deferred float.
3. **AD-11 / AD-14** — Env lint; sandbox keys rejected in production; `KycGate` live-only; `merchant_id` (+ `livemode` if shared DB) mandatory; Staff PIN hashed; API key Argon2id/scrypt + prefix + revoke; PII policy for logs **and** jobs/traces; KYC docs encrypted + operator ACL; `ams` counsel accept before live PII.
4. **AD-11** — NFR-12 audit tuple + append-only consented text hash; STOP≠ANNULER and shared-phone inside the versioned body.

## Disposition

| Finding | Action |
| --- | --- |
| Inbound Rail / `rail.poll` | Autofix into AD-8 / AD-13 — Rule, not Deferred |
| T+1 custody | Autofix into AD-7 — one sentence |
| Sandbox × KycGate / tenant isolation | Autofix into AD-11 / AD-14 |
| PII jobs/traces/KYC/residency | Autofix policy + counsel gate; KYC checklist stays Deferred |
| HMAC window + rotation | Autofix into AD-8 |
| Staff PIN / API-key KDF | Autofix into AD-11 |
| Consent NFR-12 tuple | Autofix into AD-11 |
| OTP / Magic Link abuse | Bind rate-limit + single-use in AD-11 |
| Webhook SSRF | Autofix into AD-8 |
| BFF vs API auth matrix | Autofix into AD-11 or conventions |
| Idempotency scope | Autofix into AD-8 |
| Copy CI, operator id, TLS/cookies, STOP worker, reversal≠payout | Autofix or conventions; low |

No finding asks to drop reminder-then-pay, to live-call `DirectDebitPort`, to add Customer wallets, or to treat FR-43 as EME KYC.
