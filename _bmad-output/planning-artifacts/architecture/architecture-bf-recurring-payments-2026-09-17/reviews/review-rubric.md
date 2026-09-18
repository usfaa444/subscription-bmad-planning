# Rubric Walker — ARCHITECTURE-SPINE.md

Spine: `architecture-bf-recurring-payments-2026-09-17/ARCHITECTURE-SPINE.md`  
Altitude: initiative (must keep independently built **features / epics** from diverging)  
Driver: PRD `prd-bf-recurring-payments-2026-09-17` (FR-1–FR-56, NFR-1–NFR-12)  
Lint: `lint_spine.py` — 0 mechanical findings  
Reviewed: 2026-09-17

## Verdict

**Needs revision before feature/epic handoff.** The money, consent, rail, and CountryPack invariants are real and mostly enforceable. The spine is not yet a closed substrate: it contradicts itself on where the portal BFF lives, cites a missing `AD-18`, leaves STOP ≠ Cancel unbound, and is silent on persistence, inbound SMS, KYC blobs, and Staff-PIN storage — forks two feature teams will invent incompatibly.

## Checklist scorecard

| # | Check | Result |
| --- | --- | --- |
| 1 | Fixes real divergence points for the level below; misses none | **Partial.** Load-bearing forks (hexagonal monolith, one Consent SM, reminder-then-pay, rails, ledger, idempotency, take-rate/KYC, tenancy, deploy target) are fixed. Misses: BFF placement, inbound SMS, persistence/migrations, KYC object storage, STOP ≠ Cancel, Staff PIN hash, public retrieve shape. |
| 2 | Every AD Rule is enforceable and actually prevents its stated divergence | **Partial.** Most Rules are testable. Failures: AD-4 cites nonexistent `AD-18`; AD-11 Prevents “reuse of reminder-then-pay consent for Direct Debit” but the Rule only stores a text version; AD-9 “follow PRD semantics” is a pointer, not a rule; AD-1 paradigm prose fights the seed on BFF ownership. |
| 3 | Nothing under Deferred could let two units diverge if left unspecified | **Pass on listed items; fail on silences.** Deferred items that *are* named (SMS vendor, Orange counterparty, float, Moov/Wave live, multi-region, USSD, KYC checklist) correctly keep a port/gate locked. The holes are dimensions **not** decided, deferred, or questioned. |
| 4 | Named tech is verified-current (flag asserted-not-researched) | **Mostly pass.** Node 24.21.0 LTS Krypton, TypeScript 7.0.2, Nest 12.0.3, Next 16.3.5, React 19.3.0, PostgreSQL 18.6, BullMQ 6.3.4 match current public releases (2026-09). Soft flags: `@opentelemetry/sdk-node` 0.222.0 vs npmjs crawl still showing 0.221.0; `@opentelemetry/api` `1.9.x` is a range; Docker Compose `v2` is unpinned. |
| 5 | If a spec/PRD drove it, it covers that spec’s capabilities | **Mostly pass.** Capability map cites FR-1–FR-56 and NFR-1–NFR-12. Thin or ungoverned: FR-31 STOP mapping, FR-32 silence-after-success, FR-33 named SMS, FR-54 PAY→OTP→Rail-wallet sequence, NFR-6 Staff PIN storage, NFR-2 8s/weight, §14 fail-closed on rail outage, §16 retrieve Session. |
| 6 | Every initiative-owned dimension is decided, deferred, or an open question — especially the operational/environmental envelope | **Partial.** AD-14 locks Fly.io `ams`, Compose local, `local` \| `sandbox` \| `production`, OTel + JSON logs, no in-country compute assumption. Silent: persistence/ORM/migrations, production Redis product, KYC blob store, secrets/CI/backups, inbound SMS ingress, data-retention floor. |
| 7 | No placeholders; decisions not rationale; diagrams valid | **Pass with defects.** Lint clean (no TBD/TODO/template tokens). Prose is decision-shaped. Three mermaid graphs parse. Defects: dangling `AD-18`; paradigm “Next.js BFFs” vs Nest BFF in the diagram/seed; first diagram routes inbound SMS into `SmsAdapter`; ERD models `ClientRecord }|--|{ StandingAuthorization` as M:N and omits Subscription. |
| 8 | Invariants first, seed minimal | **Pass.** Paradigm → 14 ADs → conventions → stack seed → tree + ERD. Seed is cold-start sized. AD-11 and AD-13 are dense bundles, not seed leakage. Take-rate / KYC numbers correctly live in invariants (policy), not in the ERD. |

Brownfield ratification: N/A (greenfield). Parent-spine inheritance: N/A.

---

## Findings

### Critical

None. The ledger, reminder-then-pay, and hexagonal-boundary ADs would not by themselves produce two incompatible money systems.

### High

**H1 — Portal BFF ownership contradicts itself**  
Disposition: **autofix**  
Where: Design Paradigm L24 (“Next.js BFFs”); diagram `BFF` inside `apps/api Nest + workers`; Structural Seed `apps/api` “portal BFF”; AD-1 Rule.  
Two feature teams that obey every AD can still split: portal epics put a Next Route Handler / server-action BFF in `apps/portal`; API epics put Nest controllers in `apps/api`. Hexagonal dependency is then undecidable (Next importing domain vs Nest composing adapters).  
Fix: Strike “Next.js BFFs” from the paradigm. Rule: Next apps are HTTP clients only; all mutation and reads of aggregates go through `apps/api` (public `/v1` or portal BFF). Diagram and seed already say this.

**H2 — AD-4 cites missing AD-18; AD-11 does not enforce its own Prevents**  
Disposition: **autofix**  
Where: AD-4 Rule (“new `StandingAuthorization` text version (AD-18)”); AD-11 Prevents vs Rule.  
Spine ADs are AD-1–AD-14. `AD-18` exists only in `.memlog.md` (PII / consent version). AD-11 Prevents “reuse of reminder-then-pay consent for Direct Debit” but the Rule only stores a text version — storage ≠ new-version gate. A Direct Debit epic can attach to the existing SA text and still claim compliance.  
Fix: Delete the `AD-18` cite. Add to AD-11 Rule: enabling `DirectDebitPort` requires a **new** `StandingAuthorization` text version; reminder-then-pay text is not a mandate. Do not renumber live ADs.

**H3 — STOP ≠ Cancel is a safety guardrail with no Rule**  
Disposition: **autofix**  
Where: AD-13 lists verbs; AD-3 / AD-9 omit STOP; capability map FR-18, FR-29–FR-33 → AD-2, AD-13; PRD FR-31, §12 Safety.  
SMS and portal/lifecycle epics can each treat `STOP` as Subscription Cancel (or drop billing SMS). PRD forbids that. Listing the verb is not a mapping.  
Fix: One sentence on AD-13 (or AD-9): `STOP` records a non-essential-SMS preference and does **not** Cancel; transactional Invoice / reminder / failure / Receipt continue until `ANNULER`, Merchant Cancel, or Client Record remove. Disclose in consent text (FR-11).

**H4 — Persistence / query / migration layer is silent**  
Disposition: **discuss**  
Where: Stack; Structural Seed; no AD / Deferred row.  
Initiative altitude owns the write path. Two epics will pick Prisma vs Drizzle vs TypeORM vs `pg` + raw SQL and mint incompatible schemas, migration dirs, and transaction helpers — breaking AD-8’s “outbox in the same transaction as the aggregate.” Hexagonal ports do not decide the adapter’s schema tool.  
Fix: Pin one persistence adapter + migration runner in Stack (and a one-line AD-1/seed note: domain stays SQL-free). Or Deferred with a single owner and a revisit-before-first-migration condition. Do not leave it unnamed.

**H5 — Inbound SMS / no-browser confirm has no ingress contract**  
Disposition: **discuss**  
Where: AD-2 (`SmsPort` outbound only); diagram `SMS --> SMSA`; capability map FR-18, FR-29–FR-33 and FR-54.  
Outbound `SmsPort` + `sms.send` is locked. Inbound PAY / PAUSE / SOLDE / ANNULER / STOP and FR-54 PAY → OTP → Rail-wallet confirm are not. Diagram implies `SmsAdapter` owns inbound; AD-2 says ports are outbound. SMS vs checkout epics can build (a) vendor webhook → Nest HTTP → domain vs (b) bidirectional `SmsPort` vs (c) PAY that only mints a Magic Link (browser-required, dropping FR-54).  
Fix: Decide: inbound SMS is an HTTP adapter (vendor webhook) that invokes the same domain commands as Hosted Checkout; `SmsPort` stays send-only. Name the no-browser confirm as a first-class path on AD-3/AD-4 (PAY + OTP + Rail-wallet; not a platform shortcode).

**H6 — Staff PIN storage is architecture-owned and unbound**  
Disposition: **autofix**  
Where: AD-11 Rule hashes API keys only; NFR-6 “Staff PIN cannot be recovered in plaintext. Storage mechanics are architecture.”  
Portal auth and privileged-action epics can store PIN as reversible ciphertext or plaintext and still satisfy AD-11’s “required when enabled” list.  
Fix: AD-11 Rule: Staff PIN is hashed at rest, not recoverable in plaintext (same class as API keys).

**H7 — KYC document bytes have no store**  
Disposition: **discuss**  
Where: AD-10 (flag + threshold only); FR-43 upload NINEA/RCCM/ID; no port, no seed, not Deferred.  
KYC and Merchant-workspace epics will pick Postgres BYTEA vs Fly volume vs Tigris/S3 vs “metadata only.” That is a shared-data shape and an ops envelope hole.  
Fix: Add `DocumentStorePort` (or equivalent) + one production backing store, **or** Deferred: “upload UX + accepted flag only; blob store chosen at first KYC story” with an owner.

### Medium

**M1 — ClockPort vs CalendarPort split is undecidable**  
Disposition: **autofix**  
Where: AD-2 names both; AD-7 T+1 via `ClockPort`; AD-13 T+1 via `CalendarPort`.  
Payout-clock and obligation-birth epics will put holidays on different ports.  
Fix: One rule: `ClockPort` is now; `CalendarPort` is CountryPack business days / holidays. T+1 and due dates use `CalendarPort`. Or collapse to one port.

**M2 — Public retrieve / webhook payload left to invention**  
Disposition: **autofix**  
Where: Conventions lock `POST /v1/checkout/sessions` + `Idempotency-Key`; PRD §16 also requires retrieve Session and Obligation/Attempt status; memlog AD-17 “webhook payload is status+tokens only” did not survive distill.  
API and portal epics can mint `/v1/sessions` vs `/v1/checkout/sessions/:id` vs `/v1/obligations`, and put MSISDN / Orange refs on the webhook.  
Fix: Convention: `GET /v1/checkout/sessions/:id` (and documented Attempt/Obligation reads under `/v1`). Webhook `data` is status + tokens only — no Rail PIN/OTP/secrets, no full MSISDN.

**M3 — Operational envelope is incomplete (not silent, but not closed)**  
Disposition: **defer**  
Where: AD-14 + deploy diagram.  
Decided: Fly.io `ams` + Managed Postgres, Compose local, three envs, OTel + JSON logs, multi-region Deferred. Unnamed: production Redis product (diagram shows Redis on Fly; Upstash vs Fly Redis vs Redis Cloud), secrets mechanism, backups/PITR, CI, sandbox as a **separate** Fly app + Postgres (diagram implies isolate; Rule does not), rail-outage fail-closed (§14), consent/ledger retention floor (§17).  
Fix: Either one AD-14 sentence each, or named Deferred rows with owners. A whole ops concern left unnamed is the failure, not a clean spine.

**M4 — Subscription aggregate and ClientRecord–SA cardinality are seed forks**  
Disposition: **discuss**  
Where: ERD; AD-9 emits `subscription.cancelled`; no Subscription entity.  
Lifecycle vs receivables epics can create a `Subscription` table or derive it from Plan + SA + Obligations, and can model SA as M:N with ClientRecord (ERD `}|--|{`) vs the PRD “one Client Record + Plan → Standing Authorization.”  
Fix: ERD: `ClientRecord ||--o{ StandingAuthorization`. State whether Subscription is a persisted aggregate or a derived read of Plan + SA + Obligations.

**M5 — AD-9 “follow PRD semantics” does not prevent Cancel vs Session `cancelled` by itself**  
Disposition: **autofix**  
Where: AD-9 Rule.  
The distinct event name is good. Pause / Skip / Write-off / FR-52 remove side effects are “follow PRD.” Two lifecycle stories can still disagree on open Obligations after Cancel or remove.  
Fix: Lift the three PRD locks into the Rule: Cancel revokes SA and stops billing SMS, open Obligations remain until collected / Skipped / Written-off; remove Client Record is not Customer self-serve and requires Staff PIN when enabled; Write-off is Merchant-only and cannot apply to a `collected` Attempt.

**M6 — NFR-2 weight / 8s and FR-32 / FR-33 are mapped but not ruled**  
Disposition: **defer**  
Where: Capability map NFR-2 → AD-3, AD-14; FR-18, FR-29–FR-33 → AD-13.  
Checkout and SMS copy epics can ship a heavy Next checkout and chatty success SMS. Not a schema fork; still a dual-channel divergence the initiative claims to own.  
Fix: Deferred: checkout bundle / 3G 8s budget is UX + implementation; CountryPack owns named-SMS and silence-after-success copy. Or one convention row: one SMS Receipt on success; no extra thank-you mesh (SM-C1).

### Low

**L1 — `@opentelemetry/api` `1.9.x` and Docker Compose `v2` are not pins**  
Disposition: **autofix** (OTel) / **ignore** (Compose v2 as platform family)  
Lint does not catch a non-empty range. Pin `1.9.0` (or current 1.9 patch) if the stack table is the seed contract.

**L2 — `@opentelemetry/sdk-node` 0.222.0 may be one tick ahead of the public npm page**  
Disposition: **discuss**  
npmjs crawl in this pass still listed 0.221.0 (updated 2026-07-21). Memlog claims 0.222.0 verified 2026-09-17. Confirm on npm before finalize; do not treat the table as researched if the registry disagrees.

**L3 — AD-11 / AD-13 bundle many dimensions**  
Disposition: **ignore**  
Enforceable, just dense. Split only if a later epic spine needs a cite. Do not renumber.

**L4 — Memlog AD-1…AD-26 vs spine AD-1…AD-14**  
Disposition: **ignore** (except H2)  
Distill remapped IDs. Spine IDs are the contract. Downstream must cite spine AD-n, not memlog labels.

**L5 — NFR-7 “no Customer email harvest” is unspoken**  
Disposition: **autofix**  
One Auth / PII convention row. Cheap; prevents a “backup email” story from harvesting Customer email while Merchant email stays Deferred.

---

## What holds

- Paradigm (hexagonal modular monolith; Next as clients; no microservices until a new AD) is the right initiative call and matches the PRD “rendered twice, not two products.”
- AD-4 reminder-then-pay, AD-5 Orange-only live, AD-7 pass-through / no Customer wallet, AD-8 idempotency + signed webhooks, AD-10 take-rate / KYC, AD-12 status vocabulary are enforceable and prevent the divergences they name.
- Capability map is the right shape and does not drop an FR/NFR ID cluster.
- Deferred list is honest on vendor, float, second-country, USSD, and PI-SPI — those items will **not** fork units if left unspecified, because a port or “do not implement” already binds them.
- Stack pins that were checked independently (Node 24.21.0 Krypton, TS 7.0.2, Nest 12.0.3, Next 16.3.5, React 19.3.0, PostgreSQL 18.6, BullMQ 6.3.4) look researched, not invented.
- Seed is minimal. Invariants lead. No template comments, no TBD.

## Suggested apply order (Finalize)

1. Autofix: H1, H2, H3, H6, M1, M2, M5, L5.  
2. Discuss (need a one-line product/tech call): H4, H5, H7, M4.  
3. Defer with owners: M3, M6.  
4. Ignore: L3, L4, Compose `v2`.
