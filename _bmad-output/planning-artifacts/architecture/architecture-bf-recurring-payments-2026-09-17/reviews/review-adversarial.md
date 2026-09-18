# Adversarial review — architecture spine

**Artifact:** `ARCHITECTURE-SPINE.md` (bf-recurring-payments, 2026-09-17)  
**Lens:** Adversarial two-unit attack (initiative altitude → features)  
**Method:** For each hole, name two feature-level units that can each obey every adopted AD to the letter and still ship incompatible shared-data shapes, two owners of one entity, or conflicting state-mutation paths. Each pair is a missing or unenforceable AD.  
**Date:** 2026-09-17

## Verdict

**Not a closed build substrate.** The posture locks (hexagonal monolith, reminder-then-pay, vacant Direct Debit, BF CountryPack, integer XOF, named queues) would stop *product* forks. They do not stop *implementation* forks. `CheckoutSession` and `Subscription` are talked about and missing from the ER and the naming table; AD-9 states two mint paths at once; AD-8 caps debit per token, not per open Obligation; Session / Attempt / Ledger share words with three meanings; collect, cash, and T+1 settlement have no single writer. Two features built from this spine will not compose.

Do not reopen AD-1, AD-4, AD-5 (live Orange only), AD-10 take-rate percents, or AD-14 Fly.io `ams`. Close the holes below with new or tightened ADs.

## Attack pairs

Each pair is independently shippable under the current Rules. The clash is the finding.

### Pair A — Hosted Checkout (FR-10–17) vs SMS OS (FR-18, FR-29–33, FR-54)

**Obeys:** AD-3 (one machine, two skins, same `StandingAuthorization`); AD-6 (shared aggregates); ER (no `CheckoutSession`).

| | Feature: Hosted Checkout | Feature: SMS OS |
| --- | --- | --- |
| Machine state | `CheckoutSession.step` (`identify→confirm`) + `session.status` from AD-12 | No Session in the ER, so `SmsEnrollment` or `StandingAuthorization.step` |
| ClientRecord | Upserted at `POST /v1/checkout/sessions` (MSISDN + Plan ID) | Must exist before Invite SMS |
| Confirm | Session service writes SA in the Session TX | Verb handler writes SA; no Session row |

**Clash:** Dual-channel resume (start on hosted page, PAY on SMS — FR-54) has no shared row. Same merchant+MSISDN yields two ClientRecords. Two enrollments, two SAs.

### Pair B — Checkout confirm-and-collect-now vs `obligation.birth`

**Obeys:** AD-9 both clauses: “if due date is today or past at confirm, mint due now” *and* “only domain services invoked from `obligation.birth`”; PRD “mint and collect in the same Session.”

| | Feature: Checkout | Feature: Lifecycle |
| --- | --- | --- |
| Mint path | `ConfirmEnrollment` (HTTP → domain, not the queue) mints Obligation so collect can run in-session | Only the `obligation.birth` worker may call the birth service; confirm writes SA + outbox |
| Collect-now | Attempt attached to the Obligation created in the HTTP TX | Collect waits for the worker; Session TTL can expire first |

**Clash:** Both fire → two Obligations for one confirm. Only the worker fires → in-session collect is impossible. AD-9 is not one Rule.

### Pair C — SMS PAY vs Retry Inbox

**Obeys:** AD-8 (one live debit *per token*; a retry *mints a new Attempt + token*); AD-4 (collect only after confirm on *that* Attempt).

| | Feature: SMS PAY | Feature: Encaissements retry |
| --- | --- | --- |
| Token | New Attempt + token on each PAY that does not see an existing token | FR-26: retry creates a new Attempt + token |
| Live debit | `PaymentRailPort.collect` on the PAY Attempt | `PaymentRailPort.collect` on the retry Attempt |

**Clash:** Nothing forbids two *open* Attempts on one Obligation. Fatou texts PAY while Awa retries → two Orange pulls. AD-8 prevents double-charge of a token, not of a debt.

### Pair D — Subscription lifecycle vs Consent

**Obeys:** AD-9 (Paused / Cancel / `subscription.cancelled`); naming table (no `Subscription`); ER (`StandingAuthorization ||--o{ Obligation` only).

| | Feature: Lifecycle (FR-27, FR-28, FR-51–52) | Feature: Consent (FR-10–11) |
| --- | --- | --- |
| Pause / Cancel | New `Subscription` aggregate; `obligation.birth` checks `subscription.status` | `StandingAuthorization.paused_at` / `revoked_at`; webhook `data.id` = SA id |
| Event | `subscription.cancelled` uses `subscription.id` | Same event type, `standing_authorization.id` |

**Clash:** SMS PAUSE writes SA; portal Pause writes Subscription; birth sees one and misses the other. Merchants get two IDs for one relationship.

### Pair E — Portal Clients vs Session API

**Obeys:** AD-6 (“same aggregates”); PRD ClientRecord “created in the portal or implied by a Session.” No uniqueness Rule.

| | Feature: Plans & Clients (FR-6–9) | Feature: Session API (FR-14) |
| --- | --- | --- |
| Create | Portal form → `ClientRecord` with name | Session create → unnamed row `{merchant_id, msisdn}` |
| Identity | Treats `id` as the person | Treats MSISDN as the person |

**Clash:** Two rows, two SA trees, FR-52 remove cancels one tree. Receivables book splits one Customer.

### Pair F — Public Session API vs Rails / Ledger

**Obeys:** AD-12 Session statuses; AD-7 ledger events; conventions `checkout_session.<status>` / `collection_attempt.<status>` (Attempt enum unnamed).

| Word | Feature: Public API | Feature: Rails / Ledger |
| --- | --- | --- |
| `authorized` | Consent recorded, no collect (PRD FR-16) | Rail hold / Orange authorized |
| `settled` | Session status (AD-12) on a 24h object | T+1 merchant-received (AD-7) |
| `intent` | Not a Session status | First ledger / Attempt event |

**Clash:** Webhook `authorized` means consent to the API merchant and money to the portal. Session cannot both expire in 24h and later become `settled`.

### Pair G — HTTP confirm collect vs `rail.poll` vs inbound Orange

**Obeys:** AD-4 (collect after confirm); AD-13 (only *workers* must use domain services — HTTP is not a worker); AD-2 (outbound `PaymentRailPort` only; no inbound port).

| | Feature: Checkout HTTP | Feature: Rails |
| --- | --- | --- |
| Who calls `collect` | Confirm handler, in-process | `rail.poll` worker after `customer_confirmed` |
| Who writes Attempt rail status | HTTP process | Poll worker **and** an Orange IPN controller that writes the row (inbound is not a port) |

**Clash:** Two (or three) writers of `CollectionAttempt` status; two `collect` calls with the same token; outbox emits `collected` twice.

### Pair H — Rails pass-through vs Payout Clock

**Obeys:** AD-7 (“prefer pass-through”; Clock is a “target, not a guarantee”; no float); AD-2 (no `PayoutPort`); AD-13 (no payout queue).

| | Feature: Rails | Feature: Ledger / Clock |
| --- | --- | --- |
| Money | `collect` payee = Merchant `PayoutDestination` MSISDN | Collect to aggregator/platform; `settled` waits for a transfer |
| Clock | Label only; Orange `settled` = merchant already has it | Needs a job that does not exist in AD-13 |

**Clash:** One feature never posts `settled` from a payout path; the other waits forever or invents `payout.execute`.

### Pair I — Cash Mark-paid vs Payout Clock

**Obeys:** AD-5 (cash is an Attempt method); AD-7 (customer-paid immediate; “cash = already received”; merchant-received target T+1).

| | Feature: Encaissements | Feature: Payout Clock |
| --- | --- | --- |
| Cash | Attempt `collected` + ledger `settled` in one TX; Clock shows received now | Every Attempt, including cash, sits `pending settlement` until T+1 |

**Clash:** Accueil and Encaissements disagree on the same cash row. AD-7 states both “already received” and “T+1.”

### Pair J — Collect vs Ledger vs TakeRate (LedgerEntry)

**Obeys:** AD-7 (event names); AD-10 (`TakeRatePolicy` is the only *fee authority*); ER (Attempt *and* TakeRatePolicy both link to `LedgerEntry`).

| | Feature: Collect | Feature: Ledger | Feature: Take-rate |
| --- | --- | --- | --- |
| Who inserts `LedgerEntry` | Attempt service on `collected` | Projector from outbox only | Policy object writes the fee line |

**Clash:** Double fee, missing fee, or first-success incremented in two stores.

---

## Findings

Disposition: **autofix** = tighten / add an AD with no product trade-off; **discuss** = real fork the user must pick; **defer** = not needed to start the next altitude; **ignore** = seed or already locked.

### Critical

- **[critical] [autofix]** `CheckoutSession` is required by AD-3 / AD-12 / FR-14 and is absent from the ER and the naming table (Pair A; Structural Seed ER; AD-3; AD-12) — Two skins cannot share a machine that has no row. *Close:* Add `CheckoutSession` to the glossary-in-code table and the ER: `Merchant ||--o{ CheckoutSession`, `CheckoutSession }|--|| Plan`, `CheckoutSession }o--|| ClientRecord` (upserted), `CheckoutSession }o--o| StandingAuthorization`. Session is a renderer token + status projection, not a second consent type. SMS Invite and PAY attach to the same Session or to a shared `ConsentProgress` owned by domain — pick one name and use it in both adapters. Domain service `AdvanceConsent` is the only step writer.

- **[critical] [autofix]** AD-9 states two Obligation mint paths (Pair B; AD-9 Rule) — “Mint due now at confirm” and “only domain services invoked from `obligation.birth`” cannot both be exclusive. *Close:* Replace the last AD-9 sentence with: `ObligationBirth` is the only service that inserts `Obligation`. Callers: (1) confirm path, in-process, when the Plan due date is today or past and the Session will collect now; (2) `obligation.birth` worker for later cycles. `SmsPort`, HTTP adapters, and workers must not insert rows themselves. One SA confirm produces at most one first Obligation (unique `(standing_authorization_id, cycle_index)`).

- **[critical] [autofix]** AD-8 allows two live Rails debits on one Obligation (Pair C; AD-8; FR-24; FR-26) — Token uniqueness is necessary and not sufficient. *Close:* At most one `CollectionAttempt` in `{intent, authorized, customer_confirmed}` per Obligation. PAY, hosted confirm, and Retry Inbox must reuse that open Attempt or fail. A new token is minted only after the open Attempt is `failed`, `expired`, or Merchant-retry after terminal fail. Confirm `Idempotency-Key` replays the existing Attempt; it is not a second token.

### High

- **[high] [discuss]** `Subscription` is an event and a lifecycle and not an entity (Pair D; AD-9; Consistency Conventions) — Pause / Resume / Cancel / STOP-vs-ANNULER have no owner. *Close (pick one):* **(1)** Alias: `Subscription` **is** the confirmed `StandingAuthorization`; pause/revoke live on that row; `subscription.cancelled` `data.id` = SA ULID; drop a second table. **(2)** `Subscription` is the lifecycle aggregate; SA is an immutable consent snapshot; birth and portal read Subscription only. Do not leave both readings open.

- **[high] [autofix]** ClientRecord has two create owners and no natural key (Pair E; AD-6; FR-7; FR-14; FR-52) — *Close:* Unique `(merchant_id, canonical_msisdn)` via CountryPack parse. Session create and portal create call the same `UpsertClientRecord` domain service. FR-52 remove is that row (and Cancels every SA/Subscription under it).

- **[high] [autofix]** Three status vocabularies share words and have no map (Pair F; AD-7; AD-12; Events convention) — *Close:* Publish a table in the spine:

  | Layer | Allowed values | `authorized` means | `settled` means |
  | --- | --- | --- | --- |
  | Session (AD-12) | `authorized`, `collected`, `failed`, `expired`, `cancelled` | SA confirmed, no collect in this Session | **Remove from Session** (24h object; settlement is Attempt/Ledger) |
  | CollectionAttempt | `intent`, `customer_confirmed`, `authorized`, `collected`, `failed`, `expired`, `reversed` | Rail accepted / held | n/a |
  | LedgerEntry.type | AD-7 list | Rail authorized posting | Merchant-received (Rail) or immediate (cash) |

  Webhook `type` values are the Events convention only; do not reuse Session words on Attempt events without a prefix.

- **[high] [discuss]** `CollectionAttempt` rail status has three mutation paths (Pair G; AD-2; AD-4; AD-13) — *Close:* `CollectAttempt` is the only service that calls `PaymentRailPort.collect` and that writes rail fields on the Attempt. HTTP confirm, `rail.poll`, and inbound Orange IPN are adapters that invoke it. Add a driving-adapter sentence to AD-2: inbound Rail callback and inbound SMS are adapters, not ports, and must not write aggregates. Then pick **sync-in-confirm** vs **confirm-then-`rail.poll`** in one line so Checkout and Rails do not both call Orange.

- **[high] [discuss]** Pass-through vs T+1 has no money motion (Pair H; AD-7; AD-2; AD-13) — *Close:* Bind: `PaymentRailPort.collect` payee is the current `PayoutDestination`. No `PayoutPort` and no payout queue in MVP. `settled` is recorded when the Rail reports settlement (or, if the adapter cannot, when operators mark it — not a silent T+1 job). Payout Clock is visibility on those timestamps, not a transfer. Float stays Deferred.

- **[high] [autofix]** Cash vs T+1 is self-contradictory (Pair I; AD-7) — *Close:* Cash Mark-paid sets `customer_paid` and `merchant_received` in the same TX (`already received`). T+1 / `pending settlement` applies only to Rail collects. Cash still posts ledger `collected` + `settled`, 0% take-rate, no `PaymentRailPort` call.

- **[high] [autofix]** `LedgerEntry` has three writers (Pair J; AD-7; AD-10; ER) — *Close:* `Ledger` domain service is the only inserter. Collect and `TakeRatePolicy.compute` run in the Attempt’s collected TX and *request* postings; they do not `INSERT`. ER: `CollectionAttempt ||--o{ LedgerEntry`; drop `TakeRatePolicy ||--o{ LedgerEntry}` (policy is not a parent row).

### Medium

- **[medium] [autofix]** `ClockPort` and `CalendarPort` both claim T+1 (AD-2, AD-7, AD-13) — Feature Payout implements `ClockPort.nextBusinessDay`; Feature Birth implements `CalendarPort.addBusinessDays`. *Close:* `ClockPort` = `now()` (testable). `CalendarPort` = CountryPack TZ + holidays + business-day math. T+1 and due dates use `CalendarPort` only.

- **[medium] [autofix]** Outbox and BullMQ are both “the” publisher (AD-8, AD-13) — Feature API `queue.add` after commit; Feature Workers relay the outbox. *Close:* Domain writes outbox in the aggregate TX and never imports BullMQ. One relay adapter publishes each outbox row to exactly one named queue, keyed by outbox id.

- **[medium] [autofix]** `OtpPort` vs `SmsPort` vs Magic Link (AD-2, AD-11, FR-12, FR-54) — Feature Auth owns `OtpPort` for login; Feature SMS stores PAY OTP on the Attempt and sends via `SmsPort`. *Close:* `OtpPort` issues/verifies Customer identify, Merchant login, and PAY confirm (TTL 15m). Delivery is `SmsPort`. Magic Link is `OtpPort` (signed token) delivered by `SmsPort`. Rail-wallet OTP is Orange — never `OtpPort`.

- **[medium] [autofix]** CountryPack is a package but not a port (AD-1, AD-2, AD-6, seed) — Domain may import `packages/country-packs` (not in the AD-1 ban list) or hide it behind `CalendarPort`. *Close:* Domain depends on a `CountryPack` value object / port. `packages/country-packs` is the adapter. Domain does not import BF modules.

- **[medium] [discuss]** Portal BFF and `/v1` are two command surfaces (AD-1, AD-11, AD-12) — Feature Portal calls domain services (cookie + Staff PIN). Feature API later exposes `/v1/obligations/:id/write-off` (API key, no PIN). *Close:* Same domain commands for both adapters. Staff PIN is required on portal human actions when enabled; API-key callers are the Developer stand-in (PIN N/A) — state that. Shared command DTOs; `/v1` remains the only *public* contract (AD-12). BFF routes are not a second product API.

- **[medium] [autofix]** Confirm `Idempotency-Key` is a convention, not an AD-8 Rule (Pair C; conventions table) — API mints server tokens; Checkout treats the header as the Attempt token. *Close:* Promote the convention: Session create *and* Attempt confirm require `Idempotency-Key`. Domain mints Attempt tokens. The header is replay, stored as `(merchant_id, key) → session_id | attempt_id` in Postgres.

- **[medium] [autofix]** HMAC `rawBody` has no byte identity (AD-8) — Feature Webhooks signs `JSON.stringify` at send; Feature Developers replay re-serializes columns. *Close:* Persist the exact payload bytes in the outbox; sign those bytes; replay sends the same bytes. Clock-skew window for `t=` belongs here or in a security AD.

- **[medium] [autofix]** Obligation has no status machine (AD-9; FR-25, FR-28, FR-55, FR-56) — Feature Lifecycle stores `obligation.status`; Feature Ledger infers from Attempts. Skip + collect disagree. *Close:* Obligation statuses: `open`, `collected`, `skipped`, `written_off`. Transitions only via named domain services. Rail Reversal: `collected` → `open` for the reversed XOF (FR-56). Failed Attempt does not close the Obligation (FR-25).

- **[medium] [autofix]** Plan schedule has two next-due owners (AD-9, AD-13; FR-53) — Feature Plans writes `plan.next_due_on`; Feature Birth recomputes from anchor + interval. *Close:* Plan stores schedule fields (amount, interval, anchor, currency). `ObligationBirth` is the only next-due calculator. No mutable due cursor except rows it writes.

- **[medium] [autofix]** First-success and KYC totals are racy (AD-10) — Feature Collect `COUNT(*)`; Feature Merchant denormalizes `first_live_momo_at` / `month_xof`. Two in-flight first collects both take 0%; KYC trips early or late. *Close:* `merchant.first_live_momo_attempt_id` set with compare-and-set in the collected TX; reversal does not clear it. KYC month sum is `SUM(LedgerEntry)` in the CountryPack calendar month (live MoMo `collected` + cash), or a counter updated in that same TX — pick one, not both.

- **[medium] [autofix]** STOP preference has no row (AD-9, AD-13; FR-31) — Feature SMS puts `stop_at` on ClientRecord; Feature Consent puts it on SA. Invite vs Invoice after STOP diverge. *Close:* STOP lives on `ClientRecord` (per merchant–customer). All `sms.send` producers read it. SA revoke / Subscription Cancel is a different flag.

- **[medium] [autofix]** Event catalog is illustrative, not closed (conventions; AD-8; AD-9; AD-12) — Feature Webhooks emits only the listed types; Feature Lifecycle invents `subscription.paused`, `obligation.minted`. *Close:* The Events row is the closed MVP catalog. New types require a new AD (additive). No `checkout_session.settled` once Session loses `settled`.

- **[medium] [autofix]** Redis vs Postgres split-brain for idempotency (AD-1, AD-8, AD-14) — Feature API caches keys in Redis; Feature Domain uses unique indexes. *Close:* Redis is BullMQ (+ optional cache). Idempotency and tokens are Postgres unique constraints. Domain still does not import Redis.

- **[medium] [discuss]** Enrollment `method` vs every-Attempt method (AD-3, AD-5; FR-8 / FR-19) — Feature Consent stores `sa.rail`; later PAY skips method. Feature Collect re-runs method every Attempt. *Close:* SA may store a default rail; every `CollectionAttempt` chooses method at pay time (live set = Orange + cash). SMS PAY uses the default unless the Customer picks. Stubs are not offered live.

### Low

- **[low] [autofix]** ER `ClientRecord }|--|{ StandingAuthorization` invites a family-plan join (seed ER; §5 non-goal) — Feature Consent implements M2M; Feature Portal one-SA-per-client and leaves orphans. *Close:* `ClientRecord ||--o{ StandingAuthorization` and `Plan ||--o{ StandingAuthorization`. One active SA per `(client_record_id, plan_id)`.

- **[low] [ignore]** SandboxRail selection is almost closed (AD-11, AD-14) — Env + key namespace + no live creds in sandbox. Residual: reject sandbox keys in the production process and production keys in sandbox (state both directions). Not a two-feature fork if AD-11 is read strictly.

- **[low] [defer]** Inbound SMS queue name (AD-13) — HTTP driving adapter is enough for MVP. Do not add `sms.inbound` unless volume forces it. Not a hole if Pair G’s “adapters must not write aggregates” lands.

- **[low] [ignore]** Stack pins and Fly `ams` — out of this lens (version/reality reviewer). Do not spend an AD on them here.

---

## Proposed AD closures (for the parent Update)

Minimum set to make two features compose. Prefer amending Rules in place; add `AD-15+` only when the concern is new.

| ID | Action | Closes |
| --- | --- | --- |
| AD-3 | Name `CheckoutSession` / `ConsentProgress`; one step writer | Pair A |
| AD-5 | Default rail vs per-Attempt method | method discuss |
| AD-6 | Unique ClientRecord; `UpsertClientRecord` | Pair E |
| AD-7 | Cash immediate received; Rail `settled` = Rail report; no payout job; Clock = visibility | Pairs H, I |
| AD-8 | One open Attempt per Obligation; header = replay; Postgres uniqueness; persist-and-sign bytes | Pairs C, HMAC, Redis |
| AD-9 | Single `ObligationBirth` service; two *callers*; Obligation status enum; STOP on ClientRecord; Subscription alias or entity | Pairs B, D, Obligation, STOP, Plan |
| AD-10 | CAS first-success; one KYC sum method | first-success / KYC |
| AD-12 | Drop Session `settled`; status map table | Pair F |
| AD-13 | Outbox relay only; Clock vs Calendar | outbox, ports |
| **AD-15** | `CollectAttempt` sole rail writer; inbound adapters; sync-or-poll | Pair G |
| **AD-16** | `Ledger` sole `LedgerEntry` writer | Pair J |
| **AD-17** | `OtpPort` + `CountryPack` access + closed event catalog | OTP, pack, events |
| **AD-18** | Portal BFF and `/v1` share commands; PIN vs API key | BFF discuss |

## What this lens did not reopen

Reminder-then-pay, vacant `DirectDebitPort`, no Customer wallet, 2.5% after first live MoMo, KYC as payout-only gate, hashed keys, French-first CountryPack, ULID, integer XOF, Fly.io `ams`, named BullMQ queues, hexagonal modular monolith.

## Counts

| Tier | n | autofix | discuss | defer | ignore |
| --- | --- | --- | --- | --- | --- |
| critical | 3 | 3 | 0 | 0 | 0 |
| high | 7 | 5 | 2 | 0 | 0 |
| medium | 13 | 11 | 2 | 0 | 0 |
| low | 4 | 1 | 0 | 1 | 2 |
| **total** | **27** | **20** | **4** | **1** | **2** |
