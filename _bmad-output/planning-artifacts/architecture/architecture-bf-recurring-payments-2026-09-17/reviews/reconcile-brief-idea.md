---
title: Reconcile — architecture spine vs product brief + system idea
lens: reconcile-inputs
target: ARCHITECTURE-SPINE.md
inputs:
  - _bmad-output/planning-artifacts/briefs/brief-bf-recurring-payments-2026-09-17/brief.md
  - _bmad-output/planning-artifacts/briefs/brief-bf-recurring-payments-2026-09-17/addendum.md
  - docs/system-idea.md
precedence: PRD wins over system-idea (and over brief wording) on auto-debit and live rails
prd: _bmad-output/planning-artifacts/prds/prd-bf-recurring-payments-2026-09-17/prd.md
spine: _bmad-output/planning-artifacts/architecture/architecture-bf-recurring-payments-2026-09-17/ARCHITECTURE-SPINE.md
status: complete
created: 2026-09-17
---

# Reconcile: spine × brief × system idea

## Verdict

**Conditionally ready — hold dual-channel / CountryPack / PI-SPI sockets; patch quiet drops before treating the spine as reconciled.**

The spine inherits the brief’s receivables-and-consent OS, one product / two skins, reminder-then-pay, Orange-only live rail, vacant `DirectDebitPort`, and CountryPack sockets. It correctly applies the PRD override of system-idea (and brief-moat) language on automated debit and multi-rail-live. Dual-channel and PI-SPI readiness are **not diluted in the ADs**. CountryPack is **mildly diluted** by BF/XOF/French literals inside binding Rules. Several brief trust/SMS constraints never became invariants — including the MVP no-browser collect path that makes the SMS skin actually collect.

Do not “restore” system-idea auto-debit or live Moov/Wave. Do not restore brief “consent written now covers later A2A.”

## Precedence (applied)

| Topic | System idea | Brief | PRD (wins) | Spine |
| --- | --- | --- | --- | --- |
| Auto-debit | “plan to support automated debits … later” | Never market auto-debit; “consent copy written now must also cover later A2A”; “consent that stays valid later” | Reminder-then-pay only; today’s consent is **not** a PI-SPI mandate; new `StandingAuthorization` version required | AD-4, AD-11 — PRD |
| Live rails | Orange, Moov, Wave as current methods to integrate | Orange sole live; Moov/Wave designed | Orange live; Moov/Wave stubs; not live | AD-5 — PRD |
| Customer “login / configure recurring” | Customer logs in, enters info, configures recurring | Magic Link / OTP; pay-on-reminder | No Customer password; confirm each Attempt | AD-3, AD-11 — PRD |
| Offline SMS | SMS with instructions to access a **portal** | SMS is a renderer of the same machine | Invite SMS + SMS verbs + FR-54 no-browser collect | AD-3 renderer yes; **FR-54 confirm path not bound** |
| Collection preferences | Merchant sets payment-collection preferences | Customer picks wallet at **pay time** | Plan ≠ Rail; method step | AD-5 — PRD/brief |

Resolved overrides are **not misses**. They must stay resolved.

## What landed (do not re-decide)

- One product, two skins; one Consent State Machine `identify → consent → method → confirm` (AD-1, AD-3).
- Portal + API share one Merchant account and the same aggregates (AD-6).
- Reminder-then-pay; jobs SMS only; `PaymentRailPort.collect` after Customer confirm on that `CollectionAttempt` or Cash Mark-paid (AD-4).
- No auto-debit marketing copy (AD-4).
- Live `OrangeMoneyRail`; designed `MoovMoneyRail` / `WaveRail`; cash is a method, not a Rail (AD-5).
- Vacant `DirectDebitPort` — no MVP call, no Sandbox debit toggle (AD-2).
- Future debit must not mint a new Session type (AD-12).
- CountryPack owns locale, rails, KYC, holidays, MSISDN, wallet prefixes, currency, TZ; BF only production pack; no `if-Burkina`; country is an attribute, not an ID prefix (AD-6).
- Multi-country sockets; second-country production and `jnb` Deferred; paper shadow only.
- French-first, XOF/F CFA display, no card-network marks (conventions).
- Magic Link / OTP; Staff PIN; secrets never on Merchant servers; Sandbox isolated from live Orange (AD-11, AD-14).
- Ledger owed vs collected; pass-through; no Customer wallet; Payout Clock T+1 via `ClockPort` (AD-7).
- Idempotent Attempt tokens; signed webhooks (AD-8).
- Obligation birth only after confirmed SA; first bill = same primitive (AD-9).
- Take-rate after first live MoMo success (AD-10; brief TBD superseded by PRD 2.5%).
- SMS verbs listed; `SmsPort` hides vendor (AD-13).
- Hexagonal ports so new vendors = new adapters (AD-1, AD-2) — system-idea gateway flexibility.

## Dilution watch (requested)

### Dual-channel — ADs hold; SMS collect path is the hole

Not diluted as a product split. AD-1 prevents a second product/event-bus. AD-3 makes Hosted Checkout Page and SMS verbs renderers of one machine. AD-6 locks one Merchant account.

Residual risk is **channel completeness**, not a second system:

1. **FR-54 no-browser confirm is unbound.** Brief: feature phones are first-class; first live paths are SMS + hosted web. PRD FR-54 is **in MVP**: PAY → OTP → Rail-wallet confirm (operator USSD / in-app), without a platform shortcode. Spine AD-3 binds FR-54 and says a later USSD **renderer** may attach, then forbids a platform shortcode. That is the NEXT shortcode, not the MVP no-browser **collect**. A builder can require the Hosted Checkout Page for every Orange `collect` and still satisfy AD-4 (“Customer confirm”). The SMS skin would enroll and remind but not pay — portal/API dual-channel with a broken offline collect path.
2. Capability map line “Dual-channel system idea | API + portal/SMS” can be misread as three products. The lock is AD-3 (two skins, SMS is a renderer). Low wording risk.
3. SMS OS (FR-18, FR-29–FR-33) is mapped to AD-2/AD-13 (transport), not AD-3 (protocol). Verb **transport** vs verb **as confirm renderer** can diverge.

### CountryPack — sockets present; BF literals leaked into Rules

AD-6, AD-14 (“must not assume BF compute”), ULID-without-country-prefix, and Deferred paper-shadow are the right sockets. Dilution is literals that AD-6 said must live in pack data:

| Leak | Where | Why it matters |
| --- | --- | --- |
| Take-rate / KYC in integer **XOF** + `500000` | AD-10 | Second pack cannot reuse the policy object without editing the Rule |
| Error `message` **is French** | AD-12 | Locale should resolve from CountryPack; NFR-1 French-first is BF launch, not a forever API law |
| “CountryPack minor units **(XOF scale 0)**” | AD-13 | Example sitting inside the binding Rule; fights AD-6 “no global XOF constants” |

Payout Destination “MVP: Orange Money MSISDN” (AD-7) is acceptable seed if it stays labeled MVP. Do not let it become the port shape.

### PI-SPI readiness — not diluted; citation and moat-story traps

Readiness that **did** land:

- Named vacant `DirectDebitPort` sibling of `PaymentRailPort` (not “another collect rail”).
- Never invoked; no Sandbox toggle (AD-2, AD-4).
- Public Session type stays stable when the port attaches (AD-12).
- New `StandingAuthorization` text version required; reminder-then-pay text cannot be reused (AD-4, AD-11).
- Live PI-SPI / Request-to-Pay Deferred.

This matches PRD FR-48 / FR-11 and brief addendum “Country → scheme → Rail → Adapter.” System-idea “ready to adopt ASAP” is sockets + stable Session API, not a live adapter.

Traps:

1. **Dangling `AD-18`.** Distilled spine collapsed memlog AD-1…AD-26 into AD-1…AD-14. AD-4 still requires “a new `StandingAuthorization` text version **(AD-18)**.” Memlog AD-18 is the PII/re-consent decision, now folded into AD-4/AD-11. Builders looking up AD-18 find nothing. The invariant exists; the cite is broken.
2. Capability map “PI-SPI readiness | vacant `DirectDebitPort` | AD-2, AD-4” undersells AD-11 (re-consent) and AD-12 (Session stability). The ADs are there; the map invites “empty interface is enough.”
3. Brief differentiator “consent that stays valid later” / “copy written now must also cover later A2A” **did not land — correctly.** Restoring it would re-bundle a future mandate into live copy (PRD adversarial finding). Keep the vacant port; do not keep the old moat sentence.

## Brief / idea constraints that did not land

Quiet requirements the AD structure dropped. Product SLAs that do not cause builder divergence are noted as **altitude-correct omissions**.

### Must bind or explicitly Deferred (builder divergence)

| # | Constraint | Source | Spine today | Risk |
| --- | --- | --- | --- | --- |
| 1 | No-browser Collection Attempt confirm (PAY + OTP + operator wallet) | Brief feature-phone; PRD FR-54 | AD-3 cites FR-54; Rule only allows a later USSD renderer | SMS skin cannot collect; see dual-channel |
| 2 | SMS Receipt on collect / cash / write-off / reversal | Brief trust + success; FR-23, FR-29, FR-32 | AD-9 Invoice + 48h reminder only; no receipt event or queue contract | Invoice-only SMS OS; proof artifact missing |
| 3 | Named SMS / matching shop name on invite, invoice, receipt, checkout | Brief differentiator; FR-13, FR-33 | Unbound | Generic platform SMS; trust loop breaks |
| 4 | Silence means success — one Receipt, no thank-you extras; ping on failure/change | Brief Solution; FR-32 | Unbound. NFR-11 mapped to AD-4 (wrong home) | Dunning / celebration drift |
| 5 | SMS Invoice payload: what / whom / often / how much / by when / how to pay | Brief Scope; FR-29 | AD-9 “one SMS Invoice” with no fields | Two builders, two invoice shapes |
| 6 | Next due date shown **before** it happens (consent + Accueil) | Brief; FR-11 | Honesty copy in AD-4; due-date disclosure not in Rule | Consent/UX drift |
| 7 | STOP ≠ ANNULER; transactional SMS survive STOP | PRD FR-31; brief SMS OS | Verbs listed in AD-13; distinction is “follow PRD” | STOP implemented as Cancel |
| 8 | Family / group pay, share-pay-link, second-payer fields are NEXT | Brief Who This Serves; addendum NEXT; PRD §5 | Not in Deferred (collector mode / WhatsApp / IVR are) | Household fields sneak in via “whoever pays on Orange” |

### Landed in PRD map, not as spine invariants (accept or one-line)

- Late-fee policy on Plan (brief + system-idea penalties). Deferred only mentions compounding **copy**. Plan field can stay PRD/seed; say so.
- Failed-collection Retry Inbox as a named Merchant surface (brief Collect). Retry token math is AD-8; inbox UI is UX.
- Informal signup neighbourhood; Client Record name + MSISDN only (brief sitting). Capability map FR-1–FR-7; no AD on required fields.

### Altitude-correct omissions (not findings)

Merchant sitting **< 15 minutes**, Client register **< 60 seconds**, “ship checkout in a week,” guilds, vertical templates, thermal receipts, Mooré/Dioula. Success criteria, not cross-unit invariants.

## System-idea lines that must stay dropped

Do not add these back as ADs:

- Automated debits from customer accounts as a planned live path (vacant port + new SA only).
- Moov / Wave as live MVP methods.
- Customer password accounts or “configure recurring payment” as auto-pay setup.
- Offline flow whose only Customer action is “open a portal.”
- Merchant-locked collection rail at Plan time.

## Suggested spine patches (for distill, not applied here)

1. AD-3 or AD-4 Rule: Hosted Checkout Page **and** FR-54 no-browser path (PAY + OTP + operator Rail-wallet confirm) are equal confirm renderers; neither is a platform shortcode.
2. Replace “(AD-18)” in AD-4 with AD-11 (or restore a dedicated re-consent AD without renumbering published IDs if any downstream already cited memlog AD-18 — today only the spine cites it).
3. AD-10 / AD-12 / AD-13: amounts, KYC threshold, and Merchant error language resolve from CountryPack; BF/XOF/French are the production pack values, not Rule literals.
4. AD-9 or AD-13: transactional set is Invite, Invoice, Receipt, failure; Invoice fields locked; Receipt on `collected` / Cash Mark-paid / Write-off / Reversal; FR-32 one Receipt and stop; Merchant display name on those SMS and checkout header (FR-13/FR-33).
5. Deferred: family/group pay, share-pay-link, second-payer / beneficiary fields.
6. Capability map: dual-channel → AD-1, AD-3, AD-6; PI-SPI → AD-2, AD-4, AD-11, AD-12; NFR-11 → CountryPack copy + FR-32, not AD-4 alone.

## Findings

| Sev | Finding | Action |
| --- | --- | --- |
| **High** | FR-54 no-browser confirm unbound — dual-channel SMS skin can fail to collect | Bind in AD-3/AD-4 |
| **High** | AD-4 cites nonexistent AD-18 for PI-SPI re-consent | Point at AD-11 (or a real AD) |
| **Medium** | CountryPack diluted by XOF/French/500000 literals in AD-10, AD-12, AD-13 | Pack-data the literals |
| **Medium** | SMS Receipt, named shop chrome, silence-as-success, Invoice field contract dropped | Bind or Deferred with revisit |
| **Low** | Family/group pay absent from Deferred; STOP≠Cancel and due-date-before only via “follow PRD”; capability map undersells dual-channel / PI-SPI ADs | Add Deferred + map lines |

No critical: the spine does not re-open auto-debit, live Moov/Wave, or a second product.
