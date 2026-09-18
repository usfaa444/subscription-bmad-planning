---
stepsCompleted: [step-01-validate-prerequisites, step-02-design-epics, step-03-create-stories, step-04-final-validation]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-bf-recurring-payments-2026-09-17/prd.md
  - _bmad-output/planning-artifacts/prds/prd-bf-recurring-payments-2026-09-17/addendum.md
  - _bmad-output/planning-artifacts/architecture/architecture-bf-recurring-payments-2026-09-17/ARCHITECTURE-SPINE.md
  - _bmad-output/planning-artifacts/ux-designs/ux-bf-recurring-payments-2026-09-17/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-bf-recurring-payments-2026-09-17/EXPERIENCE.md
status: final
project_name: bf-recurring-payments
created: 2026-09-17
updated: 2026-09-17
---

# bf-recurring-payments - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for bf-recurring-payments, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR-1: Informal Merchant signup — A new Merchant can create an account with display name, MSISDN, and neighbourhood, without NINEA, RCCM, or tax ID. Signup succeeds when those three fields are present and the MSISDN is not already a Merchant; subsequent authentication uses that MSISDN (OTP or equivalent Merchant-side proof — not a Customer password).

FR-2: Sub-15-minute Merchant sitting — After signup, the happy path is Plan → Client Record → send invite with no forced extra modules. Time-to-first-invite is a measured success metric (SM-1). A new Merchant can, in one sitting, create a Plan, add the first Client Record, and trigger the first Invite SMS in under 15 minutes on a typical 3G connection.

FR-3: Portal and API on one Merchant account — A portal-created Merchant can create API keys without a second account. An API-created Merchant can open the Merchant Portal and see the same Plans, Client Records, and Obligations. Obligations created via Hosted Checkout Session appear in the portal Receivables Book.

FR-4: Staff PIN on shared devices — A Merchant can set a Staff PIN (optional to set, recommended during first sitting). When enabled, a resumed browser session prompts for the PIN before Cash Mark-paid, retry, payout-destination change, or API-key reveal (architecture also requires PIN for Cancel, Client Record remove, and Write-off).

FR-5: Immediate Developers surface and always-on Sandbox — A Merchant can create Sandbox and live API keys immediately. Sandbox keys work before any successful live collection. Live keys may collect only after Orange live credentials/onboarding are in place, but the Developers surface itself is not gated on first success. Sandbox never initiates a live Orange debit.

FR-6: Configure a Plan — A Merchant can create and edit a Plan: frequency, due dates, amount (XOF), late-fee policy, and payment collection preferences. MVP frequencies: weekly, monthly, and optional custom interval of N days. Due date is visible on the Plan and every SMS Invoice. Late-fee policy can be none, fixed XOF, or percentage of the Obligation; when set, an overdue Obligation includes the computed late fee on the next SMS Invoice. Collection preferences do not lock a Rail.

FR-7: Register a Client Record by phone — A Merchant can add a Client Record with name + MSISDN in under 60 seconds and trigger an Invite SMS. Required fields are name and MSISDN only. Duplicate MSISDN for the same Merchant is rejected with an explicit message (no silent merge). Invite SMS is sent only after the Merchant confirms send.

FR-8: Plan separated from Rail — The Customer picks the Rail at the Consent State Machine `method` step (pay time). The Plan does not permanently bind Orange. Method step is separate from consent step. Standing Authorization remains valid if the Customer pays one cycle Orange and a later cycle on another live Rail. MVP live Rail list on the Hosted Checkout Page is Orange Money plus Cash Mark-paid as a Merchant-side action (Customer is not required to “pay cash” in-page).

FR-9: First bill is the same primitive — Creating the first Obligation uses invoice + reminder + pay, not a distinct one-off product. The first SMS Invoice and the nth cycle SMS Invoice share the same fields and Consent State Machine. There is no separate “one-off invoice product” IA in MVP. Obligation birth rules are FR-53 (not at Invite SMS).

FR-10: Single Consent State Machine — Every Customer enrollment — portal invite or Hosted Checkout Session — runs identify → consent → method → confirm. A Customer cannot reach confirm without identify and consent. Portal-origin and API-origin sessions persist the same Standing Authorization object type. Abandoned sessions leave no collected Obligation and no silent debit.

FR-11: Honest reminder-then-pay consent — Consent copy describes scheduled Obligations and reminder-then-pay for as long as the Standing Authorization lasts. Hosted Checkout Page and SMS consent text include: Merchant name, amount or amount rule, frequency, next due date, that the Customer must confirm each Collection Attempt, and that they can PAUSE, REPRENDRE, or ANNULER. No UI string in MVP uses “prélèvement automatique,” “auto-debit,” “until automatic debit is enabled,” or equivalent as the live promise. Standing Authorization stores the consented text version for audit (NFR-12). Enabling a Direct Debit Port later cannot reuse this text version.

FR-12: Customer identity without passwords — A Customer identifies with Magic Link or OTP only. There is no Customer password create/reset flow. Magic Link and OTP expire after 15 minutes; replay after expiry fails. Shared-phone risk is disclosed in consent; the enrolled Customer MSISDN is the consenting party. Rail-wallet MSISDN may differ from the enrolled MSISDN (whoever confirms on Orange pays for this number). No share-pay-link, second payer, or beneficiary fields in MVP.

FR-13: Merchant-named trust chrome — Transactional SMS and the Hosted Checkout Page display the Merchant’s shop name (and neighbourhood when space allows). SMS sender/body includes the Merchant display name used at signup. Hosted Checkout Page header shows the same name; mismatch is a defect.

FR-14: Create Hosted Checkout Session — A Developer Merchant can create a Hosted Checkout Session that returns a hosted URL for a Customer MSISDN and an existing Plan ID. Session create in Sandbox returns a URL without live Orange. Session create requires an idempotency key and a Plan ID (no free-form line items). Expired Session URL does not collect. Session TTL is 24 hours.

FR-15: Hosted Checkout Page (French MoMo, no card chrome) — The Hosted Checkout Page is French-first, XOF, mobile-web, with no English-default, no `$`, and no card-network logos. First-run page language is French. Amounts display with F CFA / XOF, never `$`.

FR-16: Webhook and callback on completion — On Session completion or terminal failure, the platform sends a Webhook to the Merchant with status. Session statuses include at least: `authorized`, `collected`, `failed`, `expired`, `cancelled`. Session `cancelled` ≠ Subscription Cancel (FR-51). Webhook replay does not create a second Collection Attempt debit (FR-24). Merchant can rotate a signing secret from the Developers surface.

FR-17: Secrets never on Merchant servers — Orange Money PIN, Rail-wallet OTP, and Rail credentials never pass through the Merchant app. Hosted Checkout Session API accepts no Orange Money PIN field. Merchant-visible APIs expose Session and Collection Attempt status and tokens, not Rail secret material.

FR-18: Invite SMS — After a Client Record is registered (or a Session needs Customer action), the platform can send an Invite SMS with instructions to complete the Consent State Machine. Invite includes Merchant name, what they are joining, and Magic Link and/or OTP instructions and/or PAY verb. Invite is transactional (not stopped by STOP).

FR-19: Pay-time method choice — Alias of FR-8: the Customer chooses the Rail at the Consent State Machine `method` step.

FR-20: Reminder-then-pay only live path — The platform collects live funds only after a Customer confirmation in that Collection Attempt (or Cash Mark-paid by the Merchant). No job, API, or portal action can debit Orange without a Customer confirm for that Attempt. Scheduled jobs may send SMS Invoice / reminders; they must not debit.

FR-21: Successful Orange collect and settlement visibility — A confirm on the Hosted Checkout Page or the no-browser PAY + OTP + Rail-wallet confirm path (FR-54) can reach `collected` on Orange for a configured live Merchant. Rail-wallet confirm is operator USSD or in-app — not a platform `*shortcode#`. After `collected`, Customer-paid timestamp is set immediately (FR-35). Merchant-received moves to target T+1 visibility or `pending settlement` with last Rail status. Product acceptance does not name a required aggregator.

FR-22: Designed adapters for Moov Money and Wave — The collection interface allows additional Rails to be attached without changing Merchant Portal concepts or Hosted Checkout Session semantics. Moov Money and Wave are not live in MVP. Rail is an enumerated CountryPack field, not a hardcoded single-vendor checkout. Enabling a new Rail does not require a new Merchant account type.

FR-23: Cash Mark-paid — A Merchant can mark an open Obligation paid in cash. Cash Mark-paid closes the Obligation, writes SMS Receipt (receipt remains transactional even after STOP), and takes no Take-rate. Requires Staff PIN when enabled. Cannot be applied to an already `collected` Obligation.

FR-24: Idempotent Collection Attempts — Two confirms for the same Collection Attempt token result in at most one Orange debit. A retried Attempt for the same Obligation uses a new token and is a new Attempt. Retries, double SMS, double confirm, and Webhook replay must not double-charge.

FR-25: Obligation remains on failure — A failed Collection Attempt does not cancel the Obligation or the Standing Authorization. After Orange failure, Receivables Book still shows amount due. Customer and Merchant are notified of failure (FR-29), not of cancellation.

FR-26: Failed-collection Retry Inbox — The Merchant can see failed Attempts and retry or Cash Mark-paid. Inbox lists Obligation, Customer name/MSISDN, last Rail status, timestamp. Retry creates a new Collection Attempt and sends a new reminder/PAY path.

FR-27: Basic Pause — Customer (SMS PAUSE) or Merchant can Pause a Subscription. While Paused, no new Obligations are generated; open Obligation remains until collected, Skipped, Written-off, or left due. Customer Resumes with SMS REPRENDRE; Merchant Resumes on the Client Record.

FR-28: Basic Skip — Merchant or Customer can Skip the current or next open Obligation without Cancelling the Subscription. Skipped Obligation is closed as skipped, not collected; no Take-rate. Following cycle still generates on schedule.

FR-29: Transactional SMS set — The platform sends SMS for invite, SMS Invoice, SMS Receipt, and collection failure. SMS Invoice answers what / to whom / how often / how much / by when / how to pay. Failure SMS does not claim the Subscription is cancelled. These four remain sendable after STOP (FR-31).

FR-30: Designed SMS verbs — The Customer can text PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, REPRENDRE (accents optional in matching). Surrounding sentences are French. PAY starts the current Collection Attempt confirm path. PAUSE maps to FR-27; REPRENDRE maps to Resume. SOLDE returns current open amount, due date, and Merchant name — never a platform-held Customer balance. AIDE returns the verb list, including that STOP ≠ ANNULER. ANNULER maps to FR-51. Unknown text receives a short AIDE-equivalent, not silence — except after STOP, AIDE-equivalent is sent only if the Customer originated the text.

FR-31: STOP mapping — STOP cancels further non-essential SMS. It does not Cancel the Subscription. Transactional billing SMS continue until ANNULER, Merchant Cancel (FR-51), or Client Record removal (FR-52). After STOP: no promo / tip / marketing-like SMS; SMS Invoice, PAY reminders (capped by FR-53), failure, and SMS Receipt still send. Consent text states in French that STOP ≠ ANNULER. A second STOP within 24 hours replies with ANNULER instructions only. After STOP, unsolicited AIDE broadcasts are not sent.

FR-32: Silence means success — One success does not produce a sequence of “thank you” extras in MVP. Failure, Pause, Cancel, and Rail Reversal produce SMS; success produces only the SMS Receipt.

FR-33: Named SMS — Invite, invoice, receipt, failure, AIDE, and SOLDE identify the Merchant by shop name (display name used at signup).

FR-34: Gateway-agnostic ledger — Each Collection Attempt records intent, authorized, collected, settled, failed. Write-off is FR-55. Rail Reversal is FR-56. Orange vs Cash Mark-paid share Obligation identity and differ on Collection Attempt method. Receivables Book can answer who owes what without reading Rail payloads. Ledger has no Customer-visible wallet balance. No Merchant-initiated refund of collected funds in MVP.

FR-35: Customer-paid timestamp — On Rail success, Customer-paid is shown immediately. Portal and Customer SOLDE/receipt reflect Customer-paid within the success path, not after T+1.

FR-36: Merchant-received target T+1 — Merchant-received target is T+1 business day on the Payout Clock, published as a target SLA, not a hard guarantee. Clock shows the target date using the Burkina Faso CountryPack business-day calendar. Copy labels it a target, not a guarantee.

FR-37: Pending settlement — If Merchant-received is late, the UI shows pending settlement and the last Rail status. A collected-but-not-settled Attempt cannot look like “failed.” Last Rail status string is visible to the Merchant.

FR-38: Pass-through, no Customer wallets — No Customer-facing “wallet balance” feature exists. Merchant registers a Payout Destination (Orange Money MSISDN in MVP). If legal review requires a temporary platform float, that float is operator-only, is not exposed as a Customer wallet, and must not satisfy SOLDE. Do not implement float until counsel writes the control.

FR-39: Visible Payout Clock — Merchant Portal (and Developer-visible payment objects) expose Customer-paid and Merchant-received. Both timestamps (or pending) appear on the Obligation detail.

FR-40: Take-rate after first success — First successful live Orange collect for a Merchant has 0% platform Take-rate. Each later successful live Orange collect charges 2.5% of Rail-collected XOF (integer XOF; half-up). If collected XOF = 0, Take-rate = 0. Partial Rail success: Take-rate applies to collected XOF only; remainder stays an open Obligation on a new Collection Attempt. Late fee is inside the collected amount if that amount was collected. Fee is computed at `collected`, not at `settled`. Failed settlement after `collected` does not void the Take-rate unless FR-56 Rail Reversal fires. Rail Reversal of Attempt N reverses that Attempt’s Take-rate and does not reset “first success ever.” Sandbox `collected` never increments the first-success counter and never charges Take-rate. MVP implements 2.5% as the sole live value.

FR-41: No Take-rate on failed Attempts — A failed Orange Attempt adds 0 to platform fees.

FR-42: No Take-rate on Cash Mark-paid — Cash Mark-paid fee line is 0. There is no separate per-SMS invoice to the Merchant in MVP.

FR-43: Soft KYC threshold and payout-destination change — If cumulative successful collections (live MoMo `collected` XOF plus Cash Mark-paid XOF) exceed 500,000 XOF in a calendar month, or the Merchant changes Payout Destination, the platform requires NINEA/RCCM or national ID upload before further payouts. Crossing the threshold does not delete Plans or stop SMS Invoices. Further payouts (Merchant-received credits) are blocked until documents are uploaded and accepted. Changing Payout Destination while informal triggers the same gate. Accepted may be manual review in MVP.

FR-44: Informal start — Merchants below the gate operate with name + phone + neighbourhood only. Payouts may proceed while under threshold and payout destination is unchanged.

FR-45: CountryPack skeleton — Locale, Rails, KYC rules, holidays, MSISDN parsing, wallet prefixes, and currency are represented as a CountryPack. Burkina Faso is the only production pack. Product logic does not branch on a hardcoded “Burkina” special case outside CountryPack data. A paper shadow pack for another country may exist in docs only — not a production toggle.

FR-46: MSISDN identity — Customer and Client Record identity is MSISDN + country code, not email. Email is not required for Customer signup or pay. MSISDN parsing uses CountryPack rules.

FR-47: Currency as CountryPack attribute — Amounts are XOF in the Burkina Faso pack and are not a global hardcoded currency in product rules. Display and ledger currency resolve from the Merchant’s CountryPack.

FR-48: Direct Debit Port socket (vacant) — Live PI-SPI debit and Request-to-Pay are Out of Scope. No Merchant or Customer control starts a PI-SPI debit. No Adapter call and no Sandbox toggle posts a PI-SPI debit in MVP. FR-11 live consent is not a mandate for this port.

FR-49: No live platform USSD shortcode — MVP ships SMS + hosted web only. No MVP FR depends on a live platform `*shortcode#`. NFR-8 is satisfied by SMS verbs + Hosted Checkout Page.

FR-50: Sandbox without live Orange — Developers can complete Session + Webhook flows with simulated Rail outcomes. Sandbox `collected` does not move live money and does not count as first success (FR-40). Sandbox can simulate `failed` for Failed-collection Retry Inbox testing.

FR-51: Cancel Subscription — Customer (SMS ANNULER) or Merchant can Cancel a Subscription. After Cancel: no new Obligations; Standing Authorization revoked; billing SMS stop. Open Obligations remain due until collected, Skipped, or Written-off. Platform emits a Subscription-level cancelled event distinct from Session `cancelled`. Merchant Cancel requires Staff PIN when enabled.

FR-52: Remove Client Record — A Merchant can remove a Client Record. That Cancels every Subscription with that Customer for this Merchant and stops billing SMS. Historical SMS Receipts and ledger rows remain visible to the Merchant. Removal requires Staff PIN when enabled. Customer SOLDE after removal returns that this Merchant has no open Subscription (not a wallet balance).

FR-53: Obligation birth and reminder budget — An Obligation is minted when Standing Authorization is confirmed, for the next due date on the Plan. If that due date is today or in the past, the Obligation is due immediately. Before consent, the Receivables Book shows pending consent — not an owed amount. Invite SMS does not create an Obligation. Abandoned Hosted Checkout Session creates no Obligation if consent never completed. A Hosted Checkout Session that collects now may mint and collect the current Obligation in the same Session. While Paused, no new Obligations mint. Without Standing Authorization, no Obligation mints. At mint if already due, and on each later due date: one SMS Invoice. If the Obligation is still open 48 hours after the SMS Invoice, the platform may send at most one reminder SMS. No further PAY pings for that Obligation unless the Merchant retries (FR-26). Subsequent cycles mint on the Plan schedule after each due date while the Subscription is active and not Paused.

FR-54: No-browser Collection Attempt confirm — A Customer on a feature phone can complete a Collection Attempt without opening the Hosted Checkout Page: PAY → OTP (SMS) → Rail-wallet confirm (operator USSD or in-app). This path can reach Orange `collected` (FR-21) without a browser. It is not a platform USSD shortcode (FR-49). Hosted Checkout Page remains an equal path for 3G browsers.

FR-55: Merchant Write-off — A Merchant can Write-off an open unpaid Obligation. Write-off requires Staff PIN when enabled; Customer cannot Write-off. Write-off closes the Obligation; no Take-rate; Customer gets a transactional notice. Write-off cannot apply to a `collected` Attempt (use FR-56 if the Rail reversed).

FR-56: Rail Reversal — If the live Rail reports a previously `collected` Collection Attempt reversed, the platform records a Rail Reversal. That Attempt’s Take-rate is reversed; “first success ever” is not reset. The Obligation re-opens for the reversed XOF. Customer and Merchant are notified; Payout Clock does not show Customer-paid for the reversed Attempt.

### NonFunctional Requirements

NFR-1: French-first locale — All first-run Merchant Portal, Hosted Checkout Page, SMS, and Merchant-facing API errors are French. English may exist in developer technical identifiers, not on the Hosted Checkout Page or in SMS.

NFR-2: Feature-phone and 3G viability — The Customer happy path is completable from SMS verbs plus a low-weight Hosted Checkout Page. Hosted Checkout Page first contentful interaction targets < 8 seconds on a mid-3G QA profile.

NFR-3: Idempotency and exactly-once debit — See FR-24. Webhook at-least-once delivery is allowed; debit is at-most-once per Collection Attempt token.

NFR-4: Rail status observability — Every live Attempt stores last Rail status sufficient for FR-37. Internal operators can see Adapter-level outcome without exposing secrets to Merchants.

NFR-5: Payout Clock SLA (target) — Customer-paid: immediate on Rail success. Merchant-received: T+1 business day target. Delays surface pending settlement. Not a hard guarantee against Rail outages.

NFR-6: Secret isolation — FR-17. Live API keys are revealed at create and cannot be recovered later in full. Staff PIN cannot be recovered in plaintext. Storage mechanics are architecture.

NFR-7: MSISDN and PII minimization — MSISDN is the identity key. No Customer email harvest in MVP. Retention of SMS bodies and consent text is sufficient for dispute/audit (NFR-12) and is documented in a privacy notice (French).

NFR-8: Literacy-first verbs — Numeric/SMS verbs are not a degraded mode. Hosted web is equal, not superior. NFR-8 is tested on SMS + Hosted Checkout Page, not on a future platform USSD renderer.

NFR-9: Multi-country sockets without second production pack — CountryPack + Rail Adapter + vacant Direct Debit Port. No production CI/other pack. Currency and MSISDN rules are data.

NFR-10: Webhook reliability — Signed Webhooks; Merchant can retry from the Developers surface; platform retries failed deliveries. Numeric schedule is architecture (1m / 5m / 15m / 1h / 6h / 24h then dead-letter).

NFR-11: Shame-free tone — Dunning and failure copy is private and factual. No public defaulter wall, no shaming ranking in MVP.

NFR-12: Consent and collection audit trail — Standing Authorization text version, timestamps, Collection Attempt tokens, Cash Mark-paid actor, and payout-destination changes are retained for audit.

### Additional Requirements

**Starter template (Epic 1 Story 1 impact — AD-17):** Greenfield modular monolith. Workspace: pnpm 12.4.2 + Turborepo 2.10.13. NestJS 12.0.3 ESM (`nest new` default) + TypeScript 6.0.3 + Vitest + oxlint. Next.js 16.3.5 App Router + React 19.3.0. Prisma 7.10.0 (`prisma` + `@prisma/client` 7.x — not Prisma 8 RC). PostgreSQL 17 (local `postgres:17`; Fly MPG 17). Redis 8.10.1 local; Fly Upstash Redis fixed plan in prod. BullMQ 6.3.6 + `@nestjs/bullmq` 12.0.0. Node.js 24.21.0 LTS. OpenTelemetry Node SDK 0.222.0. Structural seed: `apps/api` (NestJS public API, portal BFF, BullMQ workers), `apps/portal` (Next.js Merchant Portal), `apps/checkout` (Next.js Hosted Checkout Page), `packages/domain` (entities, ports, Consent State Machine, policies), `packages/rails` (OrangeMoneyRail, MoovMoneyRail, WaveRail, SandboxRail, vacant DirectDebit), `packages/country-packs` (BF production pack), `packages/persistence` (Prisma schema + repos). Next.js apps are browser clients only — they call `apps/api`, they are not BFFs.

- AD-1: Hexagonal modular monolith — one NestJS API + BullMQ workers. `packages/domain` contains entities, ports, and domain services only. Domain must not import Nest, Next, `pg`, Redis, or rail SDKs. Microservices out until a later spine.
- AD-2: Ports catalog — only outbound contracts are `PaymentRailPort`, vacant `DirectDebitPort` (NotImplemented — no MVP call, no Sandbox debit toggle), `SmsPort`, `WebhookDeliveryPort`, `OtpPort`, `ClockPort` / `CalendarPort`. New vendors = new adapters, same ports.
- AD-3: One Consent State Machine, two skins — Invite SMS or Hosted Checkout Session both persist the same `StandingAuthorization`. Hosted Checkout Page and SMS verbs are renderers.
- AD-4: Reminder-then-pay only — `PaymentRailPort.collect` runs only after a Customer confirm on that `CollectionAttempt`, or Merchant Cash Mark-paid. `rail.poll` and inbound Rail webhooks may refresh status only — they must not collect. `DirectDebitPort` is never invoked.
- AD-5: Rails and pay-time method — CountryPack → scheme → Rail → Adapter. Live: `OrangeMoneyRail`. Designed stubs: `MoovMoneyRail`, `WaveRail`. Cash Mark-paid is a `CollectionAttempt` method, not a Rail. Orange commercial counterparty is adapter config.
- AD-6: CountryPack and tenancy — Locale, rails, KYC rules, holidays, MSISDN parse, wallet prefixes, currency, timezone from `CountryPack`. BF only production pack. Every tenant row has `merchant_id`. Public IDs are ULID. Portal and API share one Merchant account.
- AD-7: Ledger events `intent`, `authorized`, `collected`, `settled`, `failed`, `written-off`, `reversed`. `refunded` reserved; no Merchant-initiated refund. Prefer pass-through to Payout Destination (Orange Money MSISDN). Do not implement platform float until counsel writes the control. Cash = already received.
- AD-8: Each `CollectionAttempt` has an idempotency token; at most one live debit per token. An open Obligation has at most one in-flight collect Attempt. Session create and Attempt confirm require header `Idempotency-Key`. Domain events write a Postgres outbox in the same transaction. Webhooks: HMAC-SHA256 over `timestamp.rawBody`; header `X-Webhook-Signature` (`t=` and `v1=`). Reject if timestamp skew exceeds 300 seconds. Retry 1m / 5m / 15m / 1h / 6h / 24h then dead-letter. Inbound Orange/Rail callbacks authenticate per adapter; unauthenticated rail success is ignored.
- AD-9: Only mint function is domain `ObligationBirth.mintCurrent`. Unique key: `subscription_id` + `cycle_key`. Invite SMS / abandoned Session mint none. Pause / Skip / Cancel / Resume live on `Subscription`. Subscription Cancel emits `subscription.cancelled`, distinct from Session `cancelled`. `SmsPort` must not mint.
- AD-10: `TakeRatePolicy` is the only fee authority. Rates and KYC threshold live on the policy / CountryPack — not as UI constants. `KycGate` blocks payouts (not invoicing/SMS). KYC document blobs are private object storage; not logged. Sandbox `collected` does not increment first-success, Take-rate, or `KycGate` sums.
- AD-11: Customer Magic Link or OTP, TTL 15 minutes. Merchant MSISDN OTP. Staff PIN hashed at rest (argon2id). API keys hashed at rest; live secret shown once. Sandbox uses a separate key namespace and `SandboxRail`; no live Orange credentials in sandbox env. Prod logs, job payloads, and traces mask MSISDN to last 4. Store consented text version on `StandingAuthorization`. No Customer email harvest.
- AD-12: Public prefix `/v1`. Session create: `POST /v1/checkout/sessions` + `Idempotency-Key`. Retrieve: `GET /v1/checkout/sessions/:id`. Auth: `Authorization: Bearer` live or sandbox API key. Session statuses (only on `CheckoutSession`): `authorized`, `collected`, `failed`, `expired`, `cancelled`, `settled`. Merchant-facing error `message` is French; `code` may be English identifier. Session TTL 24 hours.
- AD-13: BullMQ queues: `obligation.birth`, `reminder.dispatch`, `sms.send`, `webhook.deliver`, `rail.poll`. Store amounts as integers in CountryPack minor units (XOF scale 0) plus `currency_code`. Persist timestamps UTC; interpret due dates and T+1 via CountryPack timezone (BF: Africa/Ouagadougou) and `CalendarPort`. Inbound vendor SMS webhook hits `apps/api` and dispatches `SmsVerbHandler` in domain.
- AD-14: Production is Fly.io, primary region `ams`. Environments: `local` | `sandbox` | `production`. `livemode` is on every money row. Observability: OpenTelemetry Node SDK + structured JSON logs. Do not bind `@nestjs/observe` SaaS. Multi-region (including `jnb`) is Deferred.
- AD-15: `Subscription` is the relationship root (Merchant + ClientRecord + Plan + StandingAuthorization). `CheckoutSession` is the persisted Consent State Machine run for both skins (`origin` = `api` or `portal_invite`). Invite SMS creates a Session. `ClientRecord` is unique on `merchant_id` + normalized MSISDN. Session create may imply a ClientRecord for that pair.
- AD-16: `CollectionService` is the only writer of `CollectionAttempt` and collect/cash/reversal transitions. `LedgerService` is the only writer of `LedgerEntry`. `SettlementService` is the only writer of Merchant-received / `settled`. Portal Cash Mark-paid and SMS PAY call `CollectionService`, not the repo.
- Consistency: PRD glossary terms in code and events. Events: `checkout_session.<status>`, `collection_attempt.<status>`, `subscription.cancelled`, `obligation.written_off`, `rail.reversed`. Envelope fields: `id`, `type`, `merchant_id`, `created_at`, `data`. Error JSON: `error` with `code`, `message`, `request_id`.
- Deploy: docker-compose.yml for local (postgres:17 + Redis 8.10.1 + api/workers + portal + checkout). Sandbox process has no live Orange env.
- Deferred (do not story as MVP): live Moov/Wave, second-country production pack, SMS vendor selection (port locked), Orange commercial counterparty, legal float, live platform USSD, Laravel/Flutter samples, WhatsApp/IVR, Prisma 8, TypeScript 7, PostgreSQL 18, microservices, payment-institution license.

### UX Design Requirements

UX-DR1: Implement DESIGN.md brand-layer tokens on shadcn/ui + Tailwind for `apps/portal` and `apps/checkout` (light-only MVP). Colors: surface `#F7F4EE`, surface-raised `#FFFFFF`, ink `#1C1915`, ink-muted `#534E45`, border `#C9C1B2`, primary `#3F5A3A`, primary-foreground `#FFFFFF`, accent `#A65D28`, accent-foreground `#FFFFFF`, success-ink `#1F4D32`, late-ink `#8A3B12`, pending-ink `#5C4A28`, hold-ink `#6B3A12`, destructive-ink `#8B1E1E`, sms-bubble-in `#FFFFFF`, sms-bubble-out `#E6EDE4`. Meet contrast targets: ink on surface ≥ 12:1; ink-muted on surface ≥ 4.5:1; primary-foreground on primary ≥ 7:1; success-ink / late-ink on surface-raised ≥ 4.5:1. No neon teal, card-network red/yellow, gold-star motifs, gradient fintech meshes, `$` or USD green, English-default gray chrome, or BF flag (red/flag-green/gold/star) as chrome.

UX-DR2: Typography uses system stack only (no webfonts on checkout or SMS-adjacent pages). Roles: display 28px/650, display-sm 22px/650, body 16px/400, body-strong 16px/600, label 14px/600, meta 13px/400, sms 16px/400, verb 15px/700 letter-spacing 0.04em. Display is one hero per sitting (shop name on checkout; "Bienvenue, {name}" only on first Accueil). Amounts always read `2 000 F CFA` in Merchant/Customer UI; developers see XOF in docs/API. Never `$`. No condensed cuts; no all-caps sentences.

UX-DR3: Layout tokens — tap target 44px; gutters 16px mobile / 24px desktop; max-portal 720px; max-checkout 420px; rounded sm 8 / md 12 / lg 16 / full 9999. Portal is single-column mobile-first. Checkout is phone-shaped even on desktop; linear four steps; no sidebar. Modal stacks one level; Staff PIN replaces the confirm sheet, not stacked. First sitting is a wizard in the content column with one primary button per screen. No elevation-as-hierarchy; cards are Paper vs Raised plus hairline earth border.

UX-DR4: Brand-layer component — Button (primary): olive fill, white text, French verb labels (*Enregistrer*, *Inviter*, *Marquer payé*, *Réessayer*), min-height 44px. Button (secondary): Raised + earth border for Skip / later / dismiss.

UX-DR5: Brand-layer component — Status pill: word + simple geometric icon (circle / dash / pause bars) + status ink. Labels: En attente de consentement · Dû · En retard · Payé · Échec · En pause · Annulé · Radié. Color-only dots banned.

UX-DR6: Brand-layer component — Payout Clock two-row card: **Client payé** (immediate, success-ink or pending-ink) and **Vous recevez** (target T+1 labeled *cible, pas une garantie*, or *en attente de règlement* + last Rail status). Collected-but-not-settled must not look like Échec. Cash Mark-paid: both clocks = déjà reçu. Rail Reversal clears Client payé.

UX-DR7: Brand-layer component — Trust chrome: Checkout and SMS header show Merchant display name in display-sm and neighbourhood in meta. No card logos, no English, no `$`. Do not invent a public consumer wordmark.

UX-DR8: Brand-layer component — KYC hold banner: clay-tinted paper `#F4E4D4`, hold-ink text. States that **payouts are held**, invoicing continues. Never "compte bloqué." Shown on Accueil and Paramètres. CTA → upload NINEA/RCCM or national ID. Three document types + pending-review state.

UX-DR9: Brand-layer component — Receivables row: name, MSISDN, amount, due date, status pill. Cash Mark-paid is a clay-accent text button on open / failed rows — first-class, not a footnote. Pending-consent rows do not show amount as owed and do not offer Cash Mark-paid.

UX-DR10: Brand-layer component — Staff PIN pad: numeric, 44px keys, title *Code équipe*. Required when set for Cash Mark-paid, retry, payout change, API-key reveal, Cancel, Client Record remove, Write-off. 3 failures → wait copy, no lockout theater. Optional-to-set, offered at end of first sitting.

UX-DR11: Brand-layer component — Consent card (checkout step 2): named Merchant, amount or amount rule, frequency, next due date, reminder-then-pay sentence, STOP ≠ ANNULER disclosure, shared-phone risk. All FR-11 fields visible without a "more" accordion on a phone. Confirm checkbox is text, not a legal wall. Back does not collect.

UX-DR12: Brand-layer component — OTP field: six equal boxes (or one numeric input of 44px height), display-sm, wide tracking. No password-manager chrome labeled "password." 15 min TTL. Resend once per minute. Auto-submit when complete.

UX-DR13: Brand-layer component — Method picker: one full-width row, Orange Money wordmark-as-text (no card-network art). Selected = primary hairline. Moov/Wave hidden, not disabled-tease. Cash is Merchant-side, not a Customer checkout method.

UX-DR14: Brand-layer component — Confirm hand-off: instructs Rail-wallet confirm (operator USSD / in-app). Spinner + "Confirmez sur Orange Money." Timeout → Échec, Obligation remains. No platform `*shortcode#` tutorial.

UX-DR15: Brand-layer component — Cash Mark-paid sheet: amount (defaults to open Obligation), optional note, Staff PIN. Sends SMS Receipt. Sets Client payé; Vous recevez = déjà reçu. Cannot apply to already collected Obligation.

UX-DR16: Brand-layer component — Retry inbox row: Customer name, MSISDN, last Rail status, timestamp, *Réessayer* + *Marquer payé*. Retry mints a new Collection Attempt + reminder/PAY path; never implies a second debit of a succeeded Attempt.

UX-DR17: Brand-layer component — Empty state: display-sm sentence + one primary button. No illustration mascots. Accueil empty after signup: "Invitez un client — rien n'est dû avant son accord."

UX-DR18: Brand-layer component — SMS bubbles and verb chips: inbound Raised, outbound olive-tinted `#E6EDE4`. Verbs PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, REPRENDRE as bold chips (`typography.verb`, rounded-full). Transcript is a first-class surface in mocks and AIDE.

UX-DR19: Brand-layer component — Developers key row: live secret shown once. Sandbox keys always available. Visible Sandbox chip on sandbox checkout. Replay must not re-debit. Signing-secret rotation and webhook replay live on Développeurs.

UX-DR20: Merchant Portal IA — First sitting forced path with no extra modules: Signup → Plan → Payout Destination → Client Record + Invite SMS → Staff PIN offered. Do not open Accueil as if enrolled until those steps (except Staff PIN) are done. Resume the next missing step.

UX-DR21: Merchant Portal top-level French nav (locked): Accueil · Clients · Plans · Encaissements · Paramètres · Développeurs. On `< md`, Développeurs is a row inside Paramètres (five-tab bottom bar). On `lg+`, Développeurs is a sixth top-nav item. First sitting is full-width with no tabs until Accueil.

UX-DR22: Merchant Portal surfaces — Accueil = Receivables Book (pending consent, open Obligations, last Collection Attempt, Payout Clock summary, KYC banner if gated). Clients = Client Record list. Client Record detail = Pause / Resume / Skip / Cancel / remove (Staff PIN when enabled). Plans = list + create. Encaissements = Failed-collection Retry Inbox, Cash Mark-paid, Write-off, Payout Clock detail. Paramètres = Payout Destination, Staff PIN, KYC upload + hold banner. Développeurs = API keys, Webhooks, Sandbox, signing-secret rotation, replay.

UX-DR23: Hosted Checkout Page is a linear four-state machine (identify → consent → method → confirm). No Customer account home. Session TTL 24h. Expired Session URL: "Ce lien a expiré. Demandez un nouveau message." — no collect. Abandoned Session mints no Obligation. Back allowed; never collects.

UX-DR24: French 8th-grade, shame-free microcopy. Banned on every surface: prélèvement automatique · auto-debit · set and forget · card-network marks · English-default checkout · `$` · public ranking of late Customers · "compte bloqué" for KYC · SOLDE as wallet balance · platform USSD shortcode as the pay path. Silence after success (one SMS Receipt, no confetti / "Félicitations !!!"). Failure is factual and private ("Orange n'a pas confirmé. L'échéance reste due." — never "Fatou a refusé de payer.").

UX-DR25: Canonical SMS bodies (field-complete drafts) for Invite, SMS Invoice (six answers), Failure, SMS Receipt, AIDE, SOLDE — named with Merchant shop name; STOP ≠ ANNULER disclosed. Exact character counts remain ops/legal-open; stories must ship the six invoice answers in the first four lines when possible.

UX-DR26: State treatments — Cold load: French Skeleton, no English placeholder. Pending consent: amount not owed. En retard: late pill + late fee on next SMS Invoice, no shame ranking. Échec: Obligation remains, last Rail status visible. Pending settlement ≠ Échec. Pause: no new SMS Invoice. Session/OTP expiry copy as specified. Duplicate MSISDN: explicit reject. Offline/3G: toast "Réseau faible. Réessayez." — no silent debit. Sandbox: visible chip; fake Rail; no live Orange; does not count toward take-rate or KYC.

UX-DR27: Interaction primitives — Tap to act, ≥ 44px, no hover-only on `sm`. MSISDN-first numeric keypad, CountryPack parse, show normalized `+226 ···` on blur. OTP 6 digits, auto-submit. Staff PIN numeric pad. SMS verbs accent-insensitive; unknown inbound → short AIDE; after STOP, AIDE-equivalent only if Customer originated. One sheet deep. Banned: infinite scroll on Receivables Book (paginate), swipe-to-shame, public late leaderboard, drag-and-drop, English toggle on Checkout, card PAN fields, Customer password create, platform shortcode tutorial.

UX-DR28: Accessibility floor — WCAG 2.1 AA on Portal and Checkout where practical. French `aria-label` on every icon-only control. Status never by color alone. Focus ring visible against surface (shadcn `ring` + primary). Screen reader announces surface on navigate ("Accueil, carnet des encaissements" / "Paiement, étape consentement, 2 sur 4."). Reduce Motion: skip skeleton shimmer. Do not trap focus in marketing chrome.

UX-DR29: Responsive — `< md` bottom tabs (5) + Développeurs in Paramètres; checkout phone column 16px gutters. `md`–`lg` tabs may move to top, still one 720px column. `≥ lg` top nav six items + 720px column; cybercafé PC sitting must complete without hover. Checkout never a wide marketing split. SMS has no breakpoint. No PWA / offline-write in MVP; a failed save does not keep a ghost Obligation.

UX-DR30: Key journeys must be completable as specified — UJ-1 Awa first sitting (signup → plan → payout → client + invite → pending-consent Accueil → Staff PIN offer) under 15 minutes; Client Record < 60 seconds. UJ-2 Fatou PAY or Magic Link → identify → consent → Orange → one SMS Receipt. UJ-3 Adama Sandbox Session → French checkout → webhook `collected`/`settled` without Orange PIN on his servers. UJ-4 failed Friday: Retry Inbox or Cash Mark-paid, Obligation remains. UJ-5 KYC hold banner + upload, invoicing continues.

UX-DR31: Composition references (spine wins on conflict): `mockups/portal-first-sitting.html`, `mockups/receivables-book.html`, `mockups/hosted-checkout.html`, `mockups/sms-transcript.html`. Spine-only (no mock) surfaces: Signup, Plan, Payout Destination, Clients list, Client Record detail, Plans, Encaissements full inbox, Paramètres, Développeurs, Checkout method/confirm.

UX-DR32: Privacy notice in French documenting retention of SMS bodies and consent text for dispute/audit (NFR-7). No Customer email harvest. Shared-phone risk disclosed in consent.

### FR Coverage Map

FR-1: Epic 1 - Informal Merchant signup (name, MSISDN, neighbourhood; no tax ID)
FR-2: Epic 1 - Sub-15-minute sitting (Plan → Client Record → Invite SMS)
FR-3: Epic 6 - Same Merchant account exposes portal and API keys
FR-4: Epic 1 - Staff PIN optional setup; recommended at end of first sitting (enforced on later money actions)
FR-5: Epic 6 - Immediate Developers surface and always-on Sandbox keys
FR-6: Epic 1 - Configure a Plan (frequency, due dates, amount, late-fee, collection prefs)
FR-7: Epic 1 - Register Client Record by phone and confirm-send Invite SMS
FR-8: Epic 2 - Plan separated from Rail; method chosen at pay time
FR-9: Epic 3 - First bill uses the same invoice → remind → pay primitive
FR-10: Epic 2 - Single Consent State Machine (identify → consent → method → confirm)
FR-11: Epic 2 - Honest reminder-then-pay consent; store text version
FR-12: Epic 2 - Customer identity via Magic Link or OTP only (15 min TTL)
FR-13: Epic 2 - Merchant-named trust chrome on checkout and SMS
FR-14: Epic 6 - Create Hosted Checkout Session API (`POST /v1/checkout/sessions`)
FR-15: Epic 2 - French Hosted Checkout Page (F CFA, no card chrome, no `$`)
FR-16: Epic 6 - Signed Webhooks and signing-secret rotation
FR-17: Epic 2 - Rail secrets never on Merchant servers or Session API
FR-18: Epic 1 - Invite SMS after confirm-send (transactional)
FR-19: Epic 2 - Pay-time method choice (alias of FR-8)
FR-20: Epic 3 - Reminder-then-pay only; jobs must not debit
FR-21: Epic 3 - Live Orange collect and settlement visibility
FR-22: Epic 3 - Designed Moov Money and Wave adapter stubs (not live)
FR-23: Epic 3 - Cash Mark-paid as first-class collect
FR-24: Epic 3 - Idempotent Collection Attempts (at-most-one debit per token)
FR-25: Epic 3 - Obligation remains on failure
FR-26: Epic 5 - Failed-collection Retry Inbox
FR-27: Epic 5 - Pause / Resume Subscription
FR-28: Epic 5 - Skip current or next Obligation
FR-29: Epic 3 - Transactional SMS set (invoice, receipt, failure; invite already Epic 1)
FR-30: Epic 4 - Designed SMS verbs (PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, REPRENDRE)
FR-31: Epic 4 - STOP ≠ Cancel; transactional billing SMS continue
FR-32: Epic 3 - Silence means success (one SMS Receipt)
FR-33: Epic 3 - Named SMS (Merchant display name)
FR-34: Epic 3 - Gateway-agnostic ledger
FR-35: Epic 3 - Customer-paid timestamp immediate
FR-36: Epic 3 - Merchant-received target T+1
FR-37: Epic 3 - Pending settlement + last Rail status
FR-38: Epic 1 - Register Payout Destination (Orange MSISDN); no Customer wallets
FR-39: Epic 3 - Visible Payout Clock on Obligation detail
FR-40: Epic 3 - Take-rate after first live MoMo success (2.5%)
FR-41: Epic 3 - No Take-rate on failed Attempts
FR-42: Epic 3 - No Take-rate on Cash Mark-paid
FR-43: Epic 5 - Soft KYC threshold and payout-destination change gate
FR-44: Epic 1 - Informal start (payouts proceed under threshold)
FR-45: Epic 1 - CountryPack skeleton (BF production pack only)
FR-46: Epic 1 - MSISDN + country code identity
FR-47: Epic 1 - Currency as CountryPack attribute
FR-48: Epic 2 - Vacant Direct Debit Port (no PI-SPI debit; consent is not a mandate)
FR-49: Epic 2 - No live platform USSD shortcode
FR-50: Epic 6 - Sandbox without live Orange (`SandboxRail`)
FR-51: Epic 5 - Cancel Subscription (ANNULER / Merchant Cancel)
FR-52: Epic 5 - Remove Client Record (cancels that Merchant’s Subscriptions)
FR-53: Epic 2 - Obligation birth on Standing Authorization (pending consent ≠ owed; reminder budget implemented in Epic 3)
FR-54: Epic 4 - No-browser Collection Attempt confirm (PAY → OTP → Rail-wallet)
FR-55: Epic 5 - Merchant Write-off
FR-56: Epic 5 - Rail Reversal (re-open Obligation, claw Take-rate, do not reset first-success)

## Epic List

### Epic 1: Merchant Onboarding & First Invite
A new informal Merchant can create an account, configure a Plan and Payout Destination, add a Client Record by phone, send the first Invite SMS, and see pending consent — not an owed amount — in one sitting on a shared 3G or cybercafé device. CountryPack, MSISDN identity, and the modular-monolith sitting surfaces ship here so later epics attach to a real Merchant, not a technical scaffold.
**FRs covered:** FR-1, FR-2, FR-4, FR-6, FR-7, FR-18, FR-38, FR-44, FR-45, FR-46, FR-47
**NFRs addressed:** NFR-1 (portal French-first), NFR-2 (3G sitting), NFR-6 (Staff PIN hashed), NFR-7 (MSISDN identity, no Customer email), NFR-9 (CountryPack sockets)

### Epic 2: Consent & Hosted Checkout
The Customer completes the one Consent State Machine — identify → consent → method → confirm setup — from an Invite Magic Link or the French Hosted Checkout Page. Standing Authorization is honest reminder-then-pay (not a future mandate). Abandoned sessions mint no Obligation; confirmed consent mints the next due Obligation. Portal-origin and later API-origin sessions share the same objects.
**FRs covered:** FR-8, FR-10, FR-11, FR-12, FR-13, FR-15, FR-17, FR-19, FR-48, FR-49, FR-53
**NFRs addressed:** NFR-1 (checkout French), NFR-2 (checkout weight), NFR-8 (web equal, not superior), NFR-12 (consent text version)

### Epic 3: Reminder-then-Pay Collection
The Merchant collects an owed cycle the only live way: remind, Customer confirms, Orange Money debits — or the Merchant Cash Mark-paid. Failures leave the Obligation open. The ledger, Payout Clock, named transactional SMS, and Take-rate policy make settlement visible and economics honest. Designed Moov/Wave adapters exist as stubs; they are not live.
**FRs covered:** FR-9, FR-20, FR-21, FR-22, FR-23, FR-24, FR-25, FR-29, FR-32, FR-33, FR-34, FR-35, FR-36, FR-37, FR-39, FR-40, FR-41, FR-42
**NFRs addressed:** NFR-3 (at-most-once debit), NFR-4 (Rail status), NFR-5 (Payout Clock SLA), NFR-11 (shame-free failure copy)

### Epic 4: SMS Customer OS
SMS is a first-class OS: designed verbs, AIDE, SOLDE, STOP ≠ Cancel, and a no-browser PAY → OTP → Rail-wallet confirm path so a feature phone can finish a Collection Attempt without the Hosted Checkout Page.
**FRs covered:** FR-30, FR-31, FR-54
**NFRs addressed:** NFR-2 (feature-phone path), NFR-8 (literacy-first verbs)

### Epic 5: Receivables Operations & KYC Gates
The Merchant runs the book after the first collect: retry failed Attempts, Pause / Resume / Skip / Cancel, remove a Client Record, Write-off, and handle Rail Reversal. When monthly volume or a payout-destination change hits the soft KYC gate, payouts hold while invoicing continues.
**FRs covered:** FR-26, FR-27, FR-28, FR-43, FR-51, FR-52, FR-55, FR-56
**NFRs addressed:** NFR-11 (no shame ranking), NFR-12 (audit of write-off actor, payout-destination changes, reversals)

### Epic 6: Developer Session, Webhooks & Sandbox
The same Merchant account enables API keys immediately. A developer creates a Hosted Checkout Session, completes Sandbox collect without live Orange, and receives signed Webhooks. Sandbox never increments first-success or KYC sums.
**FRs covered:** FR-3, FR-5, FR-14, FR-16, FR-50
**NFRs addressed:** NFR-3 (idempotent Session + replay), NFR-6 (live keys shown once), NFR-10 (signed webhook retry)

## Epic 1: Merchant Onboarding & First Invite

A new informal Merchant can create an account, configure a Plan and Payout Destination, add a Client Record by phone, send the first Invite SMS, and see pending consent — not an owed amount — in one sitting on a shared 3G or cybercafé device. CountryPack, MSISDN identity, and the modular-monolith sitting surfaces ship here so later epics attach to a real Merchant, not a technical scaffold.

**FRs covered:** FR-1, FR-2, FR-4, FR-6, FR-7, FR-18, FR-38, FR-44, FR-45, FR-46, FR-47
**UX-DRs covered:** UX-DR1, UX-DR2, UX-DR3, UX-DR4, UX-DR5, UX-DR10, UX-DR12 (Merchant OTP), UX-DR17, UX-DR20, UX-DR21, UX-DR22 (Accueil/Clients/Plans/Paramètres shells), UX-DR26 (first-sitting incomplete, duplicate MSISDN, empty Accueil), UX-DR27, UX-DR28, UX-DR29, UX-DR30 (UJ-1), UX-DR31 (portal-first-sitting / receivables-book mocks), UX-DR32

### Story 1.1: Set up initial project from starter template

As a Merchant in Burkina Faso,
I want a French mobile-web portal that already knows my country, currency, and phone rules,
So that my first sitting feels local and later features attach to one Merchant account instead of a second product.

**Acceptance Criteria:**

**Given** a greenfield repository
**When** the project is set up from the architecture starter (Nest ESM `nest new` default, Next.js App Router, Prisma 7, pnpm + Turborepo) and dependencies are installed
**Then** the repo contains `apps/api` (NestJS 12 ESM public API + portal BFF + BullMQ workers), `apps/portal` (Next.js 16 App Router), `apps/checkout` (Next.js 16 App Router), `packages/domain`, `packages/rails`, `packages/country-packs`, and `packages/persistence` (Prisma 7.10.0)
**And** the workspace uses pnpm 12.x + Turborepo, TypeScript 6.0.3, Vitest, oxlint, and local Compose with `postgres:17` and Redis 8.10.1
**And** `packages/domain` contains ports only (`PaymentRailPort`, `DirectDebitPort`, `SmsPort`, `WebhookDeliveryPort`, `OtpPort`, `ClockPort` / `CalendarPort`) and does not import Nest, Next, `pg`, Redis, or rail SDKs
**And** only CountryPack seed/config is persisted in this story — Merchant, Plan, ClientRecord, and money tables are created by the first later story that needs them
**And** `docker compose` brings up api/workers, Postgres, Redis, portal, and checkout locally with a passing health check on `apps/api`

**Given** the Burkina Faso CountryPack is the only production pack
**When** a request or render resolves locale, currency, MSISDN parse, timezone, holidays, rail list, wallet prefixes, and KYC threshold
**Then** those values come from CountryPack data (French, XOF scale 0, `Africa/Ouagadougou`, BF holidays, Orange live + Moov/Wave designed, KYC 500000 XOF / calendar month)
**And** product logic does not branch on a hardcoded “Burkina” special case outside that pack
**And** amounts display as `F CFA` in Merchant UI and persist as integer minor units plus `currency_code` (FR-45, FR-47)

**Given** an unauthenticated visitor opens the Merchant Portal
**When** the first paint completes
**Then** the canvas uses DESIGN.md tokens (paper `#F7F4EE`, ink `#1C1915`, olive primary `#3F5A3A`, clay accent `#A65D28`) on shadcn/ui + Tailwind, light-only, system fonts only
**And** first-run copy is French with no English-default chrome, no `$`, and no card-network marks (NFR-1, UX-DR1, UX-DR2, UX-DR3)
**And** interactive targets are at least 44px and the column is max 720px

### Story 1.2: Informal Merchant signup with MSISDN OTP

As an informal Merchant,
I want to create an account with my shop name, phone, and neighbourhood only,
So that I can start without a tax ID and come back on a shared phone.

**Acceptance Criteria:**

**Given** a visitor who is not already a Merchant
**When** they submit display name, MSISDN, and neighbourhood and complete MSISDN OTP
**Then** a Merchant account is created with `merchant_id`, CountryPack `country_code`, and ULID public id (FR-1, FR-46)
**And** signup is not blocked for missing NINEA, RCCM, or tax ID (FR-44)
**And** no Customer or Merchant password create/reset flow is offered

**Given** an MSISDN that is already a Merchant
**When** signup is submitted
**Then** the attempt is rejected with an explicit French message
**And** a second Merchant is not created

**Given** a created Merchant
**When** they return on a later visit and complete MSISDN OTP
**Then** they authenticate as the same Merchant
**And** the MSISDN input uses a numeric keypad, CountryPack parse, and shows normalized `+226 ···` on blur (UX-DR27)
**And** the OTP field is six digits, 15-minute TTL, resend at most once per minute, auto-submit on complete, with no password-manager chrome labeled "password" (UX-DR12)

**Given** a failed save or weak 3G
**When** signup POST fails
**Then** a toast reads "Réseau faible. Réessayez."
**And** no ghost Merchant row is kept (UX-DR26, UX-DR29)

### Story 1.3: Configure the first Plan

As a Merchant in first sitting,
I want to set amount, frequency, due day, and late-fee policy in one screen,
So that I can invite a client without extra modules.

**Acceptance Criteria:**

**Given** an authenticated Merchant with no Plan
**When** first sitting resumes
**Then** the next forced step is Plan create — not Accueil, not Developers, not KYC (FR-2, UX-DR20)
**And** there is one primary French button (*Enregistrer*) (UX-DR4)

**Given** the Plan form
**When** the Merchant saves weekly or monthly frequency, due weekday or month day, integer XOF amount, late-fee policy (none | fixed XOF | percent), and collection preference
**Then** the Plan is stored on that `merchant_id` (FR-6)
**And** collection preference does not lock a Rail
**And** an optional custom interval of N days is accepted
**And** due date is visible on the saved Plan

**Given** a saved Plan
**When** the Merchant edits it later from Plans
**Then** the same fields can be updated
**And** the surface title uses `{typography.display-sm}` and amounts read `2 000 F CFA` (UX-DR2)

### Story 1.4: Register an Orange Payout Destination

As a Merchant in first sitting,
I want to register my Orange Money number as the Payout Destination,
So that later collections have a pass-through destination and I never see a Customer wallet.

**Acceptance Criteria:**

**Given** an authenticated Merchant who has a Plan and no Payout Destination
**When** first sitting resumes
**Then** the next forced step is Payout Destination (FR-38, UX-DR20)

**Given** a valid Orange Money MSISDN parsed by the CountryPack
**When** the Merchant saves it as Payout Destination
**Then** it is stored on the Merchant
**And** payouts may proceed while the Merchant is under the KYC threshold and the destination is unchanged (FR-44)
**And** no Customer-facing wallet balance feature exists
**And** SOLDE is not implemented as a platform-held balance (FR-38)

**Given** Staff PIN is already enabled
**When** the Merchant changes Payout Destination
**Then** the Staff PIN pad (*Code équipe*) is required before save (FR-4, UX-DR10)
**And** the change is retained for audit (NFR-12)

### Story 1.5: Add a Client Record and send Invite SMS

As a Merchant,
I want to add a client by name and phone and confirm-send an Invite SMS,
So that enrollment starts in under a minute without creating a debt.

**Acceptance Criteria:**

**Given** an authenticated Merchant with a Plan and Payout Destination
**When** they submit a Client Record with name + MSISDN only and confirm send
**Then** the Client Record is created unique on `merchant_id` + normalized MSISDN (FR-7, AD-15)
**And** a portal-origin `CheckoutSession` (`origin = portal_invite`) is created
**And** an Invite SMS is sent only after confirm send (FR-18)
**And** the Invite includes Merchant display name, what they are joining, Magic Link and/or OTP / PAY instructions, and that STOP does not cancel (UX-DR25)
**And** Invite SMS does not mint an Obligation (FR-53)
**And** median add + confirm-send is designed to complete in under 60 seconds (FR-7, SM-2)

**Given** a duplicate MSISDN for the same Merchant
**When** the Merchant tries to save the Client Record
**Then** the insert is rejected with an explicit French message that this number already has a Client Record
**And** no silent merge occurs (FR-7, UX-DR26)

**Given** the Invite SMS is sent
**When** delivery is recorded
**Then** the Invite is classified transactional (not marketing)
**And** the body includes Merchant display name used at signup (FR-18, FR-33)

### Story 1.6: Accueil shows pending consent, not an owed amount

As a Merchant who just invited a client,
I want Accueil to look like a receivables book with a named pending line,
So that I know the next due date without pretending money is already owed.

**Acceptance Criteria:**

**Given** first sitting Plan, Payout Destination, and at least one Client Record + Invite
**When** the Merchant reaches Accueil
**Then** French nav is Accueil · Clients · Plans · Encaissements · Paramètres · Développeurs (UX-DR21)
**And** on viewports `< md` a five-tab bottom bar is shown and Développeurs is a row inside Paramètres
**And** on `≥ lg` Développeurs is a sixth top-nav item and the sitting completes without hover (UX-DR29)
**And** first sitting does not show tabs until Accueil

**Given** a Client Record whose Standing Authorization is not confirmed
**When** Accueil renders the row
**Then** the status pill is **En attente de consentement** (word + icon + pending-ink — never color-only) (UX-DR5, UX-DR9)
**And** the amount is not shown as owed and Cash Mark-paid is not offered (FR-53, UX-DR26)
**And** the next due date from the Plan is visible
**And** the empty-owed copy is honest: nothing is due before consent

**Given** Accueil with no Client Records
**When** the Merchant opens Accueil
**Then** the empty state is a `{typography.display-sm}` sentence plus one primary button: "Invitez un client — rien n'est dû avant son accord." (UX-DR17)
**And** there is no illustration mascot and no public late ranking (NFR-11)

**Given** the Receivables Book
**When** the list is long
**Then** rows paginate — infinite scroll is not used (UX-DR27)
**And** cold load uses French `Skeleton` (no English placeholders)
**And** the screen reader announces "Accueil, carnet des encaissements" (UX-DR28)
**And** composition follows `mockups/receivables-book.html` unless the spine conflicts (UX-DR31)

### Story 1.7: Offer and store a Staff PIN

As a Merchant on a shared device,
I want to set an optional Staff PIN before I leave the machine,
So that my nephew cannot mark cash or change payout without the code.

**Acceptance Criteria:**

**Given** the Merchant has just reached Accueil after the first invite
**When** the sitting offers Staff PIN
**Then** the offer is optional and recommended, titled *Code équipe*, with 44px numeric keys (FR-4, UX-DR10)
**And** the Merchant can skip and still use Accueil
**And** the PIN is hashed at rest with argon2id and cannot be recovered in plaintext (NFR-6)

**Given** Staff PIN is enabled
**When** a resumed browser session attempts Cash Mark-paid, retry, payout-destination change, API-key reveal, Cancel, Client Record remove, or Write-off
**Then** the PIN pad is required before the action
**And** after 3 failures the UI shows wait copy with no lockout theater
**And** the PIN overlay replaces the confirm sheet (one sheet deep) (UX-DR3)

**Given** Staff PIN is not set
**When** the Merchant later opens a guarded money action
**Then** the product asks them to set a PIN (failure path from UJ-1)
**And** money actions are not left unguarded on a shared device without that prompt

### Story 1.8: French privacy notice at signup

As a Merchant,
I want a short French privacy notice before I finish signup,
So that I know MSISDN, consent text, and SMS bodies are kept for audit and not sold.

**Acceptance Criteria:**

**Given** the signup screen
**When** the Merchant completes account create
**Then** a French privacy notice is reachable and states that MSISDN, consent text, and SMS bodies are retained for dispute/audit for the life of the Merchant relationship plus a documented minimum (NFR-7, NFR-12, UX-DR32)
**And** the notice states Customer email is not harvested
**And** the notice does not require a Customer email field

**Given** production logs, job payloads, and traces
**When** an MSISDN is written
**Then** it is masked to last 4 (AD-11, NFR-7)
**And** consent bodies are not written to debug logs

## Epic 2: Consent & Hosted Checkout

The Customer completes the one Consent State Machine — identify → consent → method → confirm setup — from an Invite Magic Link or the French Hosted Checkout Page. Standing Authorization is honest reminder-then-pay (not a future mandate). Abandoned sessions mint no Obligation; confirmed consent mints the next due Obligation. Portal-origin and later API-origin sessions share the same objects.

**FRs covered:** FR-8, FR-10, FR-11, FR-12, FR-13, FR-15, FR-17, FR-19, FR-48, FR-49, FR-53
**UX-DRs covered:** UX-DR7, UX-DR11, UX-DR12, UX-DR13, UX-DR23, UX-DR24, UX-DR26 (session/OTP expiry), UX-DR27 (checkout bans), UX-DR28, UX-DR29 (checkout column), UX-DR30 (identify/consent slice of UJ-2), UX-DR31 (hosted-checkout mock)

### Story 2.1: Identify the Customer with Magic Link or OTP

As a Customer,
I want to open the named French checkout from an SMS Magic Link or enter an OTP,
So that I can continue without a password and recognize the shop I already know.

**Acceptance Criteria:**

**Given** a portal-invite `CheckoutSession` that is not expired
**When** the Customer opens the Magic Link or submits a valid OTP
**Then** they reach the identify step of the Hosted Checkout Page without creating a Customer password (FR-12)
**And** Magic Link and OTP expire after 15 minutes; replay after expiry fails with "Code expiré ou incorrect. Réessayez."
**And** OTP resend is at most once per minute (UX-DR12)

**Given** the Hosted Checkout Page
**When** any step renders
**Then** language is French-first, amounts use `F CFA`, and there are no `$`, English-default strings, or card-network logos (FR-15, NFR-1)
**And** trust chrome shows the same Merchant display name as signup and neighbourhood in meta (FR-13, UX-DR7)
**And** the page is a 420px phone column even on desktop, system fonts only, first contentful interaction targeted under 8 seconds on a mid-3G QA profile (NFR-2, UX-DR3)
**And** the screen reader announces "Paiement, étape [nom], N sur 4" (UX-DR28)
**And** no public consumer wordmark is invented

**Given** an expired Session URL (TTL 24 hours)
**When** the Customer opens it
**Then** the page shows "Ce lien a expiré. Demandez un nouveau message."
**And** no collect and no Obligation mint occur (UX-DR23)

### Story 2.2: Record honest reminder-then-pay consent

As a Customer,
I want to read and confirm a Standing Authorization that says I must confirm each collection,
So that I am not signed up for automatic debit and I know how to pause or cancel.

**Acceptance Criteria:**

**Given** a Customer who has identified
**When** they view the consent step
**Then** the Consent card shows Merchant name, amount or amount rule, frequency, next due date, that the Customer must confirm each Collection Attempt, PAUSE / REPRENDRE / ANNULER, STOP ≠ ANNULER, and shared-phone risk — all visible without a "more" accordion on a phone (FR-11, UX-DR11)
**And** confirm is an explicit text control, not a 9pt legal wall
**And** no string uses “prélèvement automatique,” “auto-debit,” “set and forget,” or “until automatic debit is enabled” (FR-11, UX-DR24)

**Given** the Customer has not identified or not consented
**When** they try to open method or confirm
**Then** they cannot reach confirm (FR-10)
**And** Back from a later step never collects (UX-DR23)

**Given** the Customer confirms consent
**When** the Standing Authorization is persisted
**Then** portal-origin and API-origin sessions store the same Standing Authorization object type (FR-10)
**And** the consented text version is stored for audit (NFR-12)
**And** that text version cannot be reused later as a Direct Debit Port mandate (FR-11, FR-48)

### Story 2.3: Choose Orange at pay time

As a Customer,
I want to pick Orange Money at the method step,
So that the Plan is not permanently tied to one Rail and I am not asked to pay cash in-page.

**Acceptance Criteria:**

**Given** a Customer who has identified and consented
**When** they reach the method step
**Then** the method picker is a separate step from consent (FR-8, FR-19)
**And** the only production Customer method is Orange Money as wordmark-as-text (UX-DR13)
**And** Moov Money and Wave are hidden, not shown as disabled teases
**And** Cash Mark-paid is not a Customer checkout method
**And** there are no share-pay-link, second-payer, or beneficiary fields; whoever confirms on Orange pays for the enrolled MSISDN (FR-12)

**Given** a selected Orange Money row
**When** the step renders
**Then** selected state is a primary hairline, not a rainbow tile
**And** the Hosted Checkout Session API and page accept no Orange Money PIN, Rail-wallet OTP, or Rail credentials (FR-17)

### Story 2.4: Confirm authorization and mint the next Obligation

As a Customer,
I want to finish confirm so my authorization is recorded and the next cycle can become due,
So that the Merchant’s book turns pending consent into a real Obligation only after I agree.

**Acceptance Criteria:**

**Given** identify + consent + method are complete and the due date is in the future
**When** the Customer confirms
**Then** Standing Authorization is active
**And** `ObligationBirth.mintCurrent` mints one Obligation for the next due date keyed by `subscription_id` + `cycle_key` (FR-53, AD-9)
**And** Accueil for that Client Record no longer shows the amount as merely pending consent

**Given** identify + consent + method are complete and the Plan due date is today or in the past
**When** the Customer confirms
**Then** the Obligation is minted due immediately (FR-53)
**And** the first bill uses the same Obligation object type as later cycles (FR-9 foundation)
**And** this story does not call `PaymentRailPort.collect` (collection is Epic 3)

**Given** no Standing Authorization
**When** mint would run
**Then** no Obligation is created (FR-53)

### Story 2.5: Abandoned sessions, vacant debit port, no platform USSD

As a Customer,
I want leaving checkout to do nothing to my money,
So that an abandoned or expired link cannot invent a debt or a silent debit.

**Acceptance Criteria:**

**Given** a CheckoutSession that never completed consent
**When** the Customer abandons the page or the Session expires
**Then** no Obligation is minted and no debit occurs (FR-10, FR-53)
**And** Accueil still shows **En attente de consentement**

**Given** the MVP surfaces
**When** a Merchant or Customer looks for a platform USSD pay path
**Then** no `*shortcode#` is documented or shipped (FR-49)
**And** NFR-8 is satisfied by SMS verbs + Hosted Checkout Page only

**Given** the vacant `DirectDebitPort`
**When** any MVP code path, Sandbox toggle, or Merchant control runs
**Then** the port is `NotImplemented` and is never invoked (FR-48, AD-2, AD-4)
**And** no PI-SPI debit or Request-to-Pay is posted
**And** today’s Standing Authorization text is not treated as a future mandate

## Epic 3: Reminder-then-Pay Collection

The Merchant collects an owed cycle the only live way: remind, Customer confirms, Orange Money debits — or the Merchant Cash Mark-paid. Failures leave the Obligation open. The ledger, Payout Clock, named transactional SMS, and Take-rate policy make settlement visible and economics honest. Designed Moov/Wave adapters exist as stubs; they are not live.

**FRs covered:** FR-9, FR-20, FR-21, FR-22, FR-23, FR-24, FR-25, FR-29, FR-32, FR-33, FR-34, FR-35, FR-36, FR-37, FR-39, FR-40, FR-41, FR-42
**UX-DRs covered:** UX-DR6, UX-DR9 (cash action), UX-DR14, UX-DR15, UX-DR24 (failure / silence), UX-DR25 (invoice / receipt / failure SMS), UX-DR26 (Payé, Échec, En retard, pending settlement), UX-DR30 (UJ-2 collect + UJ-4 cash slice)

### Story 3.1: Ledger and Collection Attempt without a Customer wallet

As a Merchant,
I want each try to collect recorded as a Collection Attempt on a gateway-agnostic ledger,
So that Accueil can answer who owes what without reading a Rail payload.

**Acceptance Criteria:**

**Given** an open Obligation
**When** a Collection Attempt is created
**Then** it has an idempotency token and records ledger events from `intent` (FR-34, AD-7, AD-8)
**And** Orange vs Cash Mark-paid share the Obligation identity and differ on Attempt method
**And** an open Obligation has at most one in-flight collect Attempt
**And** `CollectionService` is the only writer of Attempt collect/cash/reversal transitions and `LedgerService` is the only writer of `LedgerEntry` (AD-16)

**Given** the ledger
**When** a Merchant or Customer inspects balances
**Then** there is no Customer-visible wallet balance (FR-34, FR-38)
**And** `refunded` is reserved and no Merchant-initiated refund exists
**And** every money row has `livemode`

### Story 3.2: SMS Invoice, one reminder, silence after success

As a Customer,
I want one named SMS Invoice that answers what, to whom, how often, how much, by when, and how to pay,
So that I can show the message to a spouse and not be spammed.

**Acceptance Criteria:**

**Given** an Obligation minted due now, or a later cycle due date arriving
**When** `obligation.birth` / due processing runs
**Then** exactly one SMS Invoice is sent (FR-9, FR-29, FR-53)
**And** the first cycle and the nth cycle share the same SMS Invoice fields and Consent State Machine
**And** the body includes Merchant display name and the six answers in the first four lines when possible (FR-33, UX-DR25)
**And** if a late-fee policy is set and the Obligation is overdue, the next SMS Invoice includes the computed late fee and Accueil shows **En retard** (word + icon + late-ink, never color-only) (FR-6, UX-DR5, UX-DR26)
**And** `SmsPort` does not mint Obligations (AD-9)

**Given** the Obligation is still open 48 hours after the SMS Invoice
**When** `reminder.dispatch` runs
**Then** at most one reminder SMS is sent
**And** no further PAY pings go out for that Obligation unless the Merchant retries (FR-53)
**And** scheduled jobs send SMS only — they must not call `PaymentRailPort.collect` (FR-20)

**Given** a successful Collection Attempt
**When** notifications are sent
**Then** the Customer receives exactly one SMS Receipt and no celebratory extras (FR-32)
**And** Invite, invoice, receipt, and failure SMS include Merchant display name (FR-33)

### Story 3.3: Cash Mark-paid as a first-class collect

As a Merchant,
I want to mark an open Obligation paid in cash from Accueil or Encaissements,
So that Friday cash is as real as Orange and the client gets a receipt.

**Acceptance Criteria:**

**Given** an open unpaid Obligation and Staff PIN enabled
**When** the Merchant opens *Marquer payé (espèces)* and enters the PIN
**Then** the Cash Mark-paid sheet defaults amount to the open Obligation, accepts an optional note, and requires the PIN (FR-23, UX-DR15)
**And** the Obligation closes, Take-rate is 0 (FR-42), and an SMS Receipt is sent even if the Customer STOPped non-essential SMS
**And** Payout Clock shows Client payé and Vous recevez = déjà reçu (FR-35, UX-DR6)
**And** the clay-accent control is first-class on open/failed rows — not an apology footnote (UX-DR9)

**Given** an Obligation that is already `collected`
**When** the Merchant tries Cash Mark-paid
**Then** the action is rejected
**And** the ledger is unchanged

**Given** Cash Mark-paid succeeds
**When** the Merchant views Accueil
**Then** the row is **Payé** (word + icon + success-ink)
**And** no confetti or "Félicitations !!!" appears (UX-DR24)

### Story 3.4: Live Orange collect from hosted confirm

As a Customer,
I want to confirm this Collection Attempt on the Hosted Checkout Page and approve it in Orange Money,
So that the Merchant is paid without my PIN ever touching the Merchant’s app.

**Acceptance Criteria:**

**Given** an identified, consented Customer and an in-flight Collection Attempt
**When** they reach the confirm step
**Then** the page shows a hand-off spinner and "Confirmez sur Orange Money." (UX-DR14)
**And** Rail-wallet confirm is operator USSD or in-app — not a platform `*shortcode#` (FR-21)
**And** `PaymentRailPort.collect` runs only after that Customer confirm on **that** Attempt (FR-20, AD-4)

**Given** Orange reports success for a configured live Merchant
**When** the adapter returns collected
**Then** the Attempt is `collected`, Customer-paid is set immediately, and one SMS Receipt is sent (FR-21, FR-35, FR-32)
**And** the Hosted Checkout Session API still accepts no Orange PIN field (FR-17)
**And** a Session that collects now may mint and collect the current Obligation in the same Session (FR-53)
**And** inbound Orange callbacks authenticate per adapter; unauthenticated rail success is ignored (AD-8)

**Given** the confirm step times out or Orange does not confirm
**When** the hand-off ends
**Then** the Attempt is `failed`, the Obligation remains, and no silent debit occurred (UX-DR14)

**Given** `obligation.birth`, `reminder.dispatch`, `sms.send`, `webhook.deliver`, or `rail.poll` workers
**When** they run
**Then** they do not debit
**And** `rail.poll` / inbound webhooks may refresh status only (AD-4)

### Story 3.5: Idempotent attempts and honest failure

As a Customer,
I want a double tap, a second SMS, or a webhook replay to charge me at most once,
So that a failed Friday does not cancel what I owe and does not take the money twice.

**Acceptance Criteria:**

**Given** a Collection Attempt token
**When** confirm is submitted twice (page, PAY, or replay)
**Then** at most one live Orange debit occurs (FR-24, NFR-3)
**And** Session create and Attempt confirm require header `Idempotency-Key`

**Given** a failed Orange Attempt
**When** Accueil and the Customer SMS are updated
**Then** the Obligation and Standing Authorization remain (FR-25)
**And** the Customer receives a failure SMS that does not claim the Subscription is cancelled (FR-29)
**And** copy is factual: "Orange n'a pas confirmé. L'échéance reste due." — never "Fatou a refusé de payer." (NFR-11, UX-DR24)
**And** last Rail status is stored and visible to the Merchant (NFR-4)
**And** the failed Attempt adds 0 to platform fees (FR-41)

**Given** a retried collect for the same Obligation after the prior Attempt is `failed` or `expired`
**When** a new Attempt is created
**Then** it uses a new token (FR-24, AD-8)

### Story 3.6: Visible Payout Clock

As a Merchant,
I want every Obligation to show Client payé and Vous recevez separately,
So that I can tell a customer paid from whether I have received settlement.

**Acceptance Criteria:**

**Given** a `collected` Orange Attempt
**When** the Merchant opens Accueil summary or Obligation detail
**Then** **Client payé** is immediate (FR-35, FR-39, UX-DR6)
**And** **Vous recevez** shows target T+1 using CountryPack business days and is labeled *cible, pas une garantie* (FR-36, NFR-5)
**And** `SettlementService` is the only writer of Merchant-received / `settled` (AD-16)

**Given** a collected Attempt that has not settled by the target
**When** the clock renders
**Then** it shows *en attente de règlement* plus last Rail status (FR-37)
**And** the Attempt must not look like **Échec**

**Given** Cash Mark-paid
**When** the clock renders
**Then** both rows read déjà reçu

### Story 3.7: Take-rate after first live MoMo success

As a Merchant,
I want to pay nothing on my first successful Orange collect and 2.5% after that,
So that cash and failed tries stay free and Sandbox cannot create a bill.

**Acceptance Criteria:**

**Given** a Merchant with no prior successful live MoMo collect
**When** the first live Orange Attempt reaches `collected`
**Then** platform Take-rate is 0% (FR-40)
**And** `TakeRatePolicy` is the only fee authority (AD-10)

**Given** a later successful live Orange Attempt
**When** fee is computed
**Then** Take-rate is 2.5% of Rail-collected integer XOF, half-up, at `collected` not `settled` (FR-40)
**And** if collected XOF is 0, Take-rate is 0
**And** partial Rail success charges only collected XOF and leaves the remainder on a new Attempt
**And** failed settlement after `collected` does not void the Take-rate unless a Rail Reversal fires

**Given** a failed Attempt, Cash Mark-paid, or Sandbox `collected`
**When** fees are posted
**Then** Take-rate is 0 (FR-41, FR-42, FR-40)
**And** Sandbox `collected` does not increment first-success
**And** there is no separate per-SMS invoice to the Merchant

### Story 3.8: Designed Moov and Wave rail stubs

As a Merchant,
I want collection to treat Rail as a CountryPack choice,
So that a later Moov or Wave adapter can attach without a new account type.

**Acceptance Criteria:**

**Given** the CountryPack rail list
**When** adapters are composed
**Then** `OrangeMoneyRail` is the live `PaymentRailPort` implementation
**And** `MoovMoneyRail` and `WaveRail` exist as designed stubs implementing the same port and are not live (FR-22)
**And** enabling a new Rail does not require a new Merchant account type
**And** Cash Mark-paid remains a Collection Attempt method, not a Rail (AD-5)

**Given** Hosted Checkout in production
**When** the method step renders
**Then** only Orange Money is offered to the Customer
**And** product acceptance does not name a required aggregator (FR-21)

## Epic 4: SMS Customer OS

SMS is a first-class OS: designed verbs, AIDE, SOLDE, STOP ≠ Cancel, and a no-browser PAY → OTP → Rail-wallet confirm path so a feature phone can finish a Collection Attempt without the Hosted Checkout Page.

**FRs covered:** FR-30, FR-31, FR-54
**UX-DRs covered:** UX-DR18, UX-DR25 (AIDE / SOLDE / STOP), UX-DR26 (STOP / ANNULER states), UX-DR27 (verb matching), UX-DR30 (UJ-2 feature-phone path), UX-DR31 (sms-transcript mock)

### Story 4.1: Inbound verb router and AIDE

As a Customer,
I want designed French SMS verbs that work with or without accents,
So that I can run the relationship from a feature phone without installing an app.

**Acceptance Criteria:**

**Given** an inbound vendor SMS webhook to `apps/api`
**When** the body matches PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, or REPRENDRE (accent-insensitive)
**Then** `SmsVerbHandler` in domain dispatches that verb (FR-30, AD-13)
**And** surrounding sentences in replies are French
**And** AIDE returns the verb list including that STOP ≠ ANNULER and includes Merchant display name (FR-30, FR-33, UX-DR25)

**Given** unknown inbound text
**When** the Customer has not STOPped
**Then** they receive a short AIDE-equivalent, not silence (FR-30)

**Given** SMS transcript mocks and AIDE
**When** verbs are shown
**Then** they use `{typography.verb}` chips for the seven designed verbs (UX-DR18)
**And** composition follows `mockups/sms-transcript.html` unless the spine conflicts (UX-DR31)

### Story 4.2: SOLDE without a platform wallet

As a Customer,
I want to text SOLDE and see what I owe this Merchant,
So that I can check standing without a login or a platform balance.

**Acceptance Criteria:**

**Given** an active Subscription with an open Obligation
**When** the Customer texts SOLDE
**Then** the reply includes current open amount, due date, and Merchant display name (FR-30)
**And** the reply never shows a platform-held Customer balance (FR-38)
**And** Customer-paid, when relevant, reflects the success path — not T+1 (FR-35)

**Given** an active Subscription with no open Obligation
**When** the Customer texts SOLDE
**Then** the reply states there is no open amount for this Merchant
**And** it still is not a wallet balance (FR-38)

### Story 4.3: STOP is not Cancel

As a Customer,
I want STOP to quiet optional messages without ending the plan,
So that I am not stranded without bills and I am not silently unsubscribed.

**Acceptance Criteria:**

**Given** a Customer who texts STOP
**When** the preference is stored
**Then** non-essential / promo / tip SMS stop (FR-31)
**And** SMS Invoice, PAY reminders (FR-53 cap), failure, and SMS Receipt still send
**And** consent-time copy already stated in French that STOP ≠ ANNULER (FR-11, FR-31)
**And** unsolicited AIDE broadcasts are not sent after STOP
**And** Invite SMS is not re-sent after STOP unless the Merchant starts a new Consent path (FR-18, FR-31)

**Given** a Customer who already STOPped
**When** they text STOP again within 24 hours
**Then** the reply is ANNULER instructions only — no extra invoice (FR-31)

**Given** STOP is on and unknown inbound arrives
**When** the Customer originated the text
**Then** a short AIDE-equivalent may be sent
**And** if the text is not Customer-originated, AIDE-equivalent is not sent (FR-30)

### Story 4.4: PAY opens the current Collection Attempt

As a Customer with a browser,
I want PAY to start this cycle’s confirm path,
So that the SMS is the OS and checkout is an equal skin, not a different product.

**Acceptance Criteria:**

**Given** an open Obligation with an in-flight or creatable Collection Attempt
**When** the Customer texts PAY
**Then** PAY resumes the latest open Session or the in-flight Attempt for that Subscription (FR-30, AD-15)
**And** a Magic Link / Hosted Checkout confirm path is offered
**And** PAY does not debit by itself

**Given** no open Obligation and no in-flight Attempt
**When** the Customer texts PAY
**Then** they receive a short French explanation (AIDE-equivalent or SOLDE-like), not a debit

### Story 4.5: No-browser PAY, OTP, and Rail-wallet confirm

As a Customer on a feature phone,
I want to finish paying with PAY, an SMS OTP, and Orange’s own confirm,
So that I do not need the Hosted Checkout Page and there is no platform shortcode.

**Acceptance Criteria:**

**Given** an open Collection Attempt
**When** the Customer texts PAY and completes OTP over SMS
**Then** `PaymentRailPort.collect` can run after Rail-wallet confirm (operator USSD or in-app) (FR-54)
**And** this path can reach Orange `collected` without a browser (FR-21)
**And** it is not a platform USSD shortcode (FR-49)
**And** the Hosted Checkout Page remains an equal path (NFR-8)

**Given** two PAY confirms for the same Attempt token
**When** both are processed
**Then** at most one Orange debit occurs (FR-24)

### Story 4.6: SMS PAUSE and REPRENDRE

As a Customer,
I want to text PAUSE and later REPRENDRE,
So that I can stop new bills without cancelling the relationship.

**Acceptance Criteria:**

**Given** an active Subscription
**When** the Customer texts PAUSE
**Then** the Subscription is Paused and no new Obligations mint (FR-27, FR-30)
**And** any open Obligation remains until collected, Skipped, Written-off, or left due
**And** the Customer is notified; no new SMS Invoice is generated while paused (UX-DR26)

**Given** a Paused Subscription
**When** the Customer texts REPRENDRE
**Then** the Subscription Resumes and later cycles can mint again (FR-27)
**And** this is not Cancel and not STOP

### Story 4.7: SMS ANNULER ends the Subscription

As a Customer,
I want to text ANNULER to stop the plan,
So that billing SMS stop and I am not still authorized, while any open amount stays honest.

**Acceptance Criteria:**

**Given** an active or Paused Subscription
**When** the Customer texts ANNULER
**Then** Standing Authorization is revoked, no new Obligations mint, and billing SMS stop (FR-51, FR-30)
**And** open Obligations remain due until collected, Skipped, or Written-off
**And** the platform emits `subscription.cancelled`, distinct from Session `cancelled`
**And** STOP is not required and is not implied

## Epic 5: Receivables Operations & KYC Gates

The Merchant runs the book after the first collect: retry failed Attempts, Pause / Resume / Skip / Cancel, remove a Client Record, Write-off, and handle Rail Reversal. When monthly volume or a payout-destination change hits the soft KYC gate, payouts hold while invoicing continues.

**FRs covered:** FR-26, FR-27, FR-28, FR-43, FR-51, FR-52, FR-55, FR-56
**UX-DRs covered:** UX-DR8, UX-DR16, UX-DR22 (Clients detail, Encaissements), UX-DR26 (Pause, Write-off, KYC hold, En retard), UX-DR30 (UJ-4, UJ-5)

### Story 5.1: Failed-collection Retry Inbox

As a Merchant,
I want a list of failed Collection Attempts with retry and cash actions,
So that a failed Friday stays on the book until I collect or mark cash.

**Acceptance Criteria:**

**Given** one or more failed Collection Attempts
**When** the Merchant opens Encaissements
**Then** the Retry Inbox lists Obligation, Customer name/MSISDN, last Rail status, and timestamp (FR-26, UX-DR16)
**And** actions are *Réessayer* and *Marquer payé*
**And** Staff PIN is required when enabled (FR-4)

**Given** the Merchant taps *Réessayer*
**When** the prior Attempt is `failed` or `expired`
**Then** a new Collection Attempt with a new token is created and a new reminder/PAY path is sent (FR-26)
**And** a succeeded Attempt cannot be retried into a second debit (FR-24)

**Given** a failed Attempt already on the book
**When** the Merchant opens Encaissements with no live Orange
**Then** the inbox still lists the failure from the domain ledger (not a Rail payload) (FR-26, FR-34)

### Story 5.2: Merchant Pause and Resume on the Client Record

As a Merchant,
I want to Pause or Resume a Subscription from the Client Record,
So that I can honor a break without waiting for the Customer to text.

**Acceptance Criteria:**

**Given** a Client Record with an active Subscription
**When** the Merchant Pauses from Client Record detail
**Then** no new Obligations mint and the status pill can show **En pause** (FR-27, UX-DR5)
**And** Staff PIN is required when enabled
**And** this uses the same Subscription lifecycle as SMS PAUSE

**Given** a Paused Subscription
**When** the Merchant Resumes on the Client Record
**Then** later cycles can mint again
**And** there is no public defaulter wall or ranking (NFR-11)

### Story 5.3: Skip the current or next Obligation

As a Merchant or Customer,
I want to Skip this cycle without cancelling the plan,
So that one missed week does not end the relationship or charge a fee.

**Acceptance Criteria:**

**Given** a current or next open Obligation
**When** Merchant or Customer Skips it
**Then** the Obligation closes as skipped, not collected, with Take-rate 0 (FR-28)
**And** the following cycle still generates on schedule
**And** the Subscription and Standing Authorization remain

**Given** Merchant Skip on the portal
**When** Staff PIN is enabled
**Then** the PIN is required
**And** Skip uses the secondary button pattern (UX-DR4)

### Story 5.4: Merchant Cancel Subscription

As a Merchant,
I want to Cancel a Subscription from the Client Record,
So that I can stop new bills and revoke authorization without erasing open amounts.

**Acceptance Criteria:**

**Given** a Subscription
**When** the Merchant Cancels it
**Then** no new Obligations mint, Standing Authorization is revoked, and billing SMS stop (FR-51)
**And** open Obligations remain due until collected, Skipped, or Written-off
**And** `subscription.cancelled` is emitted and is distinct from Session `cancelled`
**And** Staff PIN is required when enabled
**And** the status pill can show **Annulé**

### Story 5.5: Remove a Client Record

As a Merchant,
I want to remove a Client Record,
So that that person is no longer on my roster and their subscriptions with me end.

**Acceptance Criteria:**

**Given** a Client Record with one or more Subscriptions
**When** the Merchant removes it
**Then** every Subscription with that Customer for this Merchant is Cancelled and billing SMS stop (FR-52)
**And** historical SMS Receipts and ledger rows remain visible to the Merchant
**And** Staff PIN is required when enabled
**And** Customer SOLDE after removal returns that this Merchant has no open Subscription

### Story 5.6: Merchant Write-off

As a Merchant,
I want to Write-off an open unpaid Obligation,
So that I can close a debt I will not collect without calling it paid.

**Acceptance Criteria:**

**Given** an open unpaid Obligation
**When** the Merchant Writes it off from Encaissements with Staff PIN if enabled
**Then** the Obligation closes, Take-rate is 0, and the Customer gets a transactional notice (FR-55)
**And** the Customer cannot Write-off
**And** the status pill can show **Radié**
**And** Cash Mark-paid actor / Write-off actor is retained for audit (NFR-12)

**Given** a `collected` Attempt
**When** the Merchant tries Write-off
**Then** the action is rejected
**And** they are directed to Rail Reversal if the Rail reversed (FR-55, FR-56)

### Story 5.7: Rail Reversal

As a Merchant,
I want a reversed Orange collect to reopen the Obligation and undo that attempt’s fee,
So that the book stays honest and my first-success flag is not wiped.

**Acceptance Criteria:**

**Given** a previously `collected` Collection Attempt
**When** the live Rail reports it reversed
**Then** the platform records a Rail Reversal (`rail.reversed`) (FR-56)
**And** that Attempt’s Take-rate is reversed and “first success ever” is not reset (FR-40, FR-56)
**And** the Obligation re-opens for the reversed XOF
**And** Customer and Merchant are notified
**And** Payout Clock does not show Client payé for the reversed Attempt (UX-DR6)

### Story 5.8: Soft KYC hold and document upload

As a Merchant,
I want to see why payouts are held and upload NINEA, RCCM, or national ID,
So that invoicing continues and I am not silently frozen.

**Acceptance Criteria:**

**Given** cumulative live MoMo `collected` XOF plus Cash Mark-paid XOF exceeds 500,000 XOF in the CountryPack calendar month
**When** the `KycGate` evaluates
**Then** further payouts (Merchant-received credits) are blocked until documents are uploaded and accepted (FR-43)
**And** Plans are not deleted and SMS Invoices continue
**And** Accueil and Paramètres show the KYC banner: **Paiements retenus — les factures continuent.** — never "compte bloqué." (UX-DR8, UX-DR24)
**And** Sandbox `collected` does not increment the KYC sum (AD-10)

**Given** an informal Merchant who changes Payout Destination
**When** the destination is saved
**Then** the same KYC gate triggers (FR-43)
**And** Staff PIN is required when enabled

**Given** the Paramètres upload
**When** the Merchant submits NINEA, RCCM, or national ID
**Then** three document types and a pending-review state are shown (UX-DR8)
**And** accepted may be a manual review flag in MVP
**And** KYC blobs are private object storage and are not logged (AD-10)
**And** payouts resume after accepted

## Epic 6: Developer Session, Webhooks & Sandbox

The same Merchant account enables API keys immediately. A developer creates a Hosted Checkout Session, completes Sandbox collect without live Orange, and receives signed Webhooks. Sandbox never increments first-success or KYC sums.

**FRs covered:** FR-3, FR-5, FR-14, FR-16, FR-50
**UX-DRs covered:** UX-DR19, UX-DR21 (Développeurs placement), UX-DR26 (Sandbox chip), UX-DR30 (UJ-3)

### Story 6.1: API keys on the same Merchant account

As a Developer Merchant,
I want Sandbox and live API keys on the same account I already use in the portal,
So that I do not open a second company and I can start before my first live collect.

**Acceptance Criteria:**

**Given** a portal-created Merchant
**When** they open Développeurs
**Then** they can create Sandbox and live API keys without a second account (FR-3, FR-5)
**And** the Developers surface is available immediately and is not gated on first live success
**And** Sandbox keys work before any successful live collection

**Given** a newly created live API key
**When** the secret is shown
**Then** it is shown once, stored hashed, and cannot be recovered later in full (NFR-6, UX-DR19)
**And** Staff PIN is required to reveal/create when enabled (FR-4)
**And** Sandbox uses a separate key namespace; live Orange credentials are absent from the sandbox env (AD-11, AD-14)

**Given** an API-created Merchant
**When** they open the Merchant Portal
**Then** they see the same Plans, Client Records, and Obligations (FR-3)

**Given** viewports `< md`
**When** the Merchant wants keys
**Then** Développeurs is reachable as a row inside Paramètres (UX-DR21)

### Story 6.2: Create and retrieve a Hosted Checkout Session

As a Developer Merchant,
I want `POST /v1/checkout/sessions` to return a hosted URL for a Customer MSISDN and Plan ID,
So that my app can redirect the Customer without hosting Orange chrome.

**Acceptance Criteria:**

**Given** a valid Bearer sandbox or live API key, `Idempotency-Key`, Customer MSISDN, and existing Plan ID
**When** the client calls `POST /v1/checkout/sessions`
**Then** a `CheckoutSession` is created (`origin = api`) and a hosted URL is returned (FR-14, AD-12)
**And** Session create in Sandbox returns a URL without live Orange
**And** free-form line items are rejected — Plan ID is required
**And** retrieve works via `GET /v1/checkout/sessions/:id`
**And** Merchant-facing `error.message` is French; `code` may be an English identifier (NFR-1)

**Given** an expired Session (TTL 24 hours)
**When** the Customer opens the URL or the client collects
**Then** the Session does not collect (FR-14)

**Given** Session create for an MSISDN + Merchant pair without a Client Record
**When** the Session is created
**Then** a ClientRecord may be implied for that pair (AD-15)
**And** the resulting Obligation appears in the portal Receivables Book (FR-3)

### Story 6.3: SandboxRail without live Orange

As a Developer Merchant,
I want Sandbox to finish identify → consent → method → confirm with fake Rail outcomes,
So that I can test `collected` and `failed` without moving live money.

**Acceptance Criteria:**

**Given** Sandbox keys and `SandboxRail`
**When** a Customer completes hosted checkout in `livemode=false`
**Then** simulated `collected` or `failed` can be produced (FR-50)
**And** no live Orange debit occurs
**And** Sandbox `collected` does not increment first-success, Take-rate, or KYC sums (FR-40, AD-10)
**And** a visible Sandbox chip is shown on the checkout page (UX-DR26)

**Given** Sandbox `failed`
**When** the Merchant opens the Failed-collection Retry Inbox
**Then** the simulated failure is listed for inbox testing (FR-50)

### Story 6.4: Signed Webhooks, retry, and replay

As a Developer Merchant,
I want signed Webhooks when a Session completes and a way to rotate the secret and replay,
So that my app can trust events and a replay cannot debit twice.

**Acceptance Criteria:**

**Given** a Session that reaches a terminal or settlement status
**When** the outbox worker `webhook.deliver` runs
**Then** a Webhook is POSTed with event envelope `id`, `type`, `merchant_id`, `created_at`, `data` (FR-16, AD-8)
**And** Session statuses include `authorized`, `collected`, `failed`, `expired`, `cancelled`, `settled`
**And** Session `cancelled` ≠ Subscription Cancel
**And** the signature is HMAC-SHA256 over `timestamp.rawBody` in `X-Webhook-Signature` (`t=` and `v1=`)
**And** receivers may reject timestamp skew > 300 seconds

**Given** a failed delivery
**When** the platform retries
**Then** the schedule is 1m / 5m / 15m / 1h / 6h / 24h then dead-letter (NFR-10)
**And** the Merchant can retry from Développeurs

**Given** a Merchant replay of a `collected` Webhook
**When** the replay is sent
**Then** no second Collection Attempt debit occurs (FR-16, FR-24)
**And** the Merchant can rotate the signing secret from Développeurs (FR-16, UX-DR19)

### Story 6.5: Obligations from Sessions appear in the portal book

As a Merchant who also ships an app,
I want hosted-checkout Obligations to show on Accueil with the same Payout Clock,
So that portal and API are one receivables OS.

**Acceptance Criteria:**

**Given** a Hosted Checkout Session that collected in Sandbox or live
**When** the Merchant opens Accueil
**Then** the Obligation and Payout Clock objects match what the Webhook described (FR-3, FR-39)
**And** Orange PIN never appeared on Merchant-visible APIs (FR-17)

**Given** live keys
**When** the Merchant has no Orange live credentials/onboarding
**Then** live collect is blocked
**And** the Developers surface and Sandbox keys still work (FR-5)
