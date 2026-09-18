---
title: Sarati
status: final
created: 2026-09-17
updated: 2026-09-18
---

# PRD: Sarati

*Burkina Faso–first receivables-and-consent platform. Product name locked as **Sarati**; primary domain **sarati.net**.*

## Product identity

| | |
| --- | --- |
| **Product name** | **Sarati** |
| **Etymology** | Dioula/Jula — *sarati* means agreement / contract / terms |
| **Primary domain** | **sarati.net** (purchased by the product owner) |
| **Market** | Burkina Faso first; multi-country / multi-currency architecture later |
| **Planning repo** | https://github.com/usfaa444/subscription-bmad-planning |
| **Local folder slug** | `bf-recurring-payments` (document content and titles use Sarati) |

## 0. Document Purpose

This is the founder launch PRD for **Sarati**. It is written for internal planning and is the source of product truth for UX, architecture, and epics. Structure: Glossary-anchored vocabulary, named user journeys, features with globally numbered functional requirements (FR-1…), globally numbered cross-cutting non-functional requirements (NFR-1…), and success metrics that cite those IDs. Capabilities only — transport, adapter counterparties, and storage live in `addendum.md`. It builds on `_bmad-output/planning-artifacts/briefs/brief-bf-recurring-payments-2026-09-17/` and `docs/system-idea.md`; it does not replace the brief as historical record. Product-owner decisions that closed brief open questions are locked here and in `.memlog.md`.

## 1. Vision

Build a platform in Burkina Faso that lets any business offer subscriptions and recurring collection without assembling identity, invoices, dunning, mobile-money quirks, receipts, and a ledger from scratch. Today there is no simple way to do this. Offline Merchants chase clients on WhatsApp and in person. App teams rebuild a billing stack for every product. Existing aggregators sell one-shot pay-ins, not a living who-owes-what book, a standing Customer authorization, SMS-first enrollment, or cash as an equal collection method.

The product is a **receivables-and-consent OS**, rendered twice — not two products. Businesses with websites or apps start a Hosted Checkout Session from their app; the Customer completes a French hosted page; the Merchant receives a Webhook. Businesses without a web app — weekly cleaners, tutors, schools, gyms, mosques — register on the Merchant Portal, set a Plan, register Clients by phone, and the platform sends an SMS that takes the Customer through the same Consent State Machine. One loop: identify → consent → method → confirm.

Recurring here means **customer-authorized pay-on-reminder**, never a merchant-initiated wallet pull. The live Collection Primitive is invoice → SMS → Customer confirms → Orange Money debit → Merchant credit (or Cash Mark-paid). Merchant APIs stay Session-shaped so a later Direct Debit Port can attach **after a new Standing Authorization** — today’s consent is not that mandate. Launch Burkina Faso first. Architect from day one for multiple countries and currencies via CountryPack — West Africa, then the continent, then globally — without shipping a second-country production pack in MVP.

## 1.1 Why Now

Aggregators already sell one-shot Orange/Moov pay-ins in Burkina Faso. They do not ship a Receivables Book, Standing Authorization, SMS-first enrollment, or cash as an equal method. Card-token “recurring” does not match BF MoMo, where every live debit is payer-approved in-session.

PI-SPI launched 30 September 2025; participant connection is still completing through 2026. Live merchant-initiated mandate debit is not the 2026 MVP bet. Keep the Hosted Checkout Session API stable; do not treat today’s consent as that mandate. Near-term edge is reminder-then-pay execution — French SMS, informal Merchant sitting — not a proprietary Rail. Citations: `addendum.md` §6.

## 2. Target User

### 2.1 Jobs To Be Done

- **Functional (Merchant):** Stop chasing weekly or monthly money without hiring a collector. See who is late. Get paid on the agreed day, or mark cash, with proof.
- **Functional (Customer):** Stay in good standing with a trusted provider without walking to a kiosk every cycle.
- **Functional (Developer Merchant):** Ship checkout in a week, not a quarter — Session → hosted URL → Webhook, sandbox first.
- **Emotional:** Shame-free private reminders instead of public chasing. Named SMS that matches the shop. A receipt that looks official. A Payout Clock that says “Customer paid” and “Merchant received.”
- **Social:** The Merchant looks organized. The Customer can show one SMS to a spouse or parent.
- **Contextual:** Shared Androids, cybercafé PCs, feature phones, 3G, French-first, MSISDN as identity, XOF, cash still common.

### 2.2 Non-Users (v1)

- Card-first or English-first checkout operators.
- Merchants who need live Moov Money or Wave collection on day one.
- Merchants who need a native mobile app.
- Institutions that require a payment-institution or e-money license from this platform.
- NGO / government / institutional payers, B2B supplier credit, tontine products.
- Customers expecting silent auto-debit or a consumer Super-App wallet.
- Second-country production Merchants (Côte d’Ivoire may exist only as a paper CountryPack shadow).

### 2.3 Key User Journeys

- **UJ-1. Awa enrolls her first weekly client from a cybercafé PC.**
  - **Persona + context:** Awa, weekly cleaner in Ouaga, keeps names in a notebook. She will not install an app. She wants the first Client enrolled in under a minute and to stop chasing Friday cash by WhatsApp.
  - **Entry state:** Unauthenticated. Shared PC or her Android. No tax ID. Neighbourhood known.
  - **Path:** (1) Creates a Merchant account with name, phone, neighbourhood. (2) Sets a weekly Plan: amount, due weekday, late-fee policy, collection preference. Registers a Payout Destination (Orange Money MSISDN). (3) Adds a Client Record: name + MSISDN. (4) Platform sends the Invite SMS. (5) Accueil (Receivables Book) shows **pending consent** and the next due date — not an owed Obligation until Fatou confirms. (6) After consent she sees the Obligation, then Cash Mark-paid or waits for Orange.
  - **Climax:** First Client Record enrolled and first Invite SMS sent in one sitting under 15 minutes; Client Record row took under 60 seconds.
  - **Resolution:** She has a Plan, a Client Record, pending or confirmed Standing Authorization, and a visible next due date. API keys exist on the same account if she ever wants them.
  - **Edge case:** Nephew uses the same phone — she sets a Staff PIN before leaving the machine.
  - **Capabilities:** Merchant sitting, informal KYC, Plan + Client Record, Invite SMS, Staff PIN, Payout Destination, one account for portal + API. → FR-1, FR-2, FR-3, FR-4, FR-6, FR-7, FR-18, FR-23, FR-38, FR-44, FR-53.

- **UJ-2. Fatou pays this cycle from one French SMS.**
  - **Persona + context:** Fatou, Awa’s client. Feature phone or 3G Android. No email. She will show the SMS to her husband. She pays if the shop name matches the woman who cleans.
  - **Entry state:** Unauthenticated. Holds an SMS named as Awa’s shop. No Customer password.
  - **Path:** (1) Reads SMS Invoice: what, to whom, how often, how much, by when, how to pay. (2) Replies `PAY` or opens the Magic Link. (3) Identifies via Magic Link or OTP. (4) Confirms Standing Authorization and sees the next due date. (5) Picks Orange Money at pay time (MVP live Rail). (6) Confirms the Collection Attempt on the Hosted Checkout Page **or**, on a feature phone, via PAY + OTP + Rail-wallet confirm (operator USSD or in-app — not a platform shortcode). (7) Receives SMS Receipt.
  - **Climax:** She knows she is in good standing — SMS Receipt, shop name matches, amount matches.
  - **Resolution:** Obligation collected or Cash Mark-paid later. Silence after success. She can send `SOLDE`, `PAUSE`, `REPRENDRE`, `AIDE`, `ANNULER`. `STOP` ends non-essential SMS and is **not** Cancel; billing SMS continue until she sends `ANNULER` or Awa Cancels / removes the Client Record — disclosed in consent.
  - **Edge case:** Orange fails — Obligation remains; she gets a failure SMS; Awa sees the Failed-collection Retry Inbox and can retry or Cash Mark-paid. Whoever confirms on Orange pays for Fatou’s enrolled MSISDN; no second-payer fields.
  - **Capabilities:** Consent State Machine, reminder-then-pay, Orange collect, no-browser confirm, SMS verbs, receipt, no double-charge, Cancel. → FR-10, FR-11, FR-12, FR-13, FR-15, FR-19, FR-20, FR-21, FR-24, FR-25, FR-27, FR-29, FR-30, FR-31, FR-32, FR-33, FR-51, FR-54.

- **UJ-3. Adama the developer ships hosted checkout in a week.**
  - **Persona + context:** Adama builds a BF SaaS in Ouaga. He wants Stripe-shaped docs, French XOF, no card chrome, sandbox without live Orange.
  - **Entry state:** Same Merchant account type as Awa. Developers surface available immediately. Sandbox always on.
  - **Path:** (1) Enables API keys. (2) Creates a Hosted Checkout Session for a Customer MSISDN + Plan. (3) Redirects the Customer to the Hosted Checkout Page. (4) Customer completes identify → consent → method → confirm. (5) His app receives a Webhook with status. Orange Money PIN never touches his servers.
  - **Climax:** First Sandbox Session completes; then first live Orange collect posts `collected` and later `settled` without a second integration.
  - **Resolution:** His Merchant account shows the same Obligation and Payout Clock Awa would see in the Receivables Book. Take-rate starts only after this Merchant’s first successful MoMo collection ever.
  - **Edge case:** He replays the Webhook — idempotent Collection Attempt token prevents a second debit.
  - **Capabilities:** Hosted Checkout Session API, Webhook, Sandbox, secrets stay off Merchant servers, idempotency, Orange collect, Take-rate. → FR-3, FR-5, FR-14, FR-15, FR-16, FR-17, FR-21, FR-24, FR-35, FR-36, FR-37, FR-40, FR-50.

- **UJ-4. Awa recovers a failed Friday collect.**
  - **Persona + context:** Same Awa. Orange timed out or Fatou cancelled the Rail-wallet confirm (operator USSD or in-app — not a platform shortcode).
  - **Entry state:** Authenticated in Merchant Portal (Staff PIN if shared device). Obligation still open.
  - **Path:** (1) Retry inbox shows the failed Collection Attempt and last Rail status. (2) She retries (new Attempt, same Obligation) or marks cash when Fatou pays in hand. (3) Fatou gets failure SMS, then receipt if cash or later Orange succeeds.
  - **Climax:** Receivables Book stays honest — owed until collected, skipped, or Written-off (Merchant action, Staff PIN; not Customer self-serve).
  - **Resolution:** Payout Clock shows Customer-paid only after Rail success or Cash Mark-paid; Merchant-received follows settlement rules (cash = already received).
  - **Capabilities:** Failure default, Failed-collection Retry Inbox, cash first-class, settlement visibility, Staff PIN, Write-off. → FR-4, FR-23, FR-25, FR-26, FR-29, FR-35, FR-36, FR-37, FR-55.

- **UJ-5. Awa hits the soft KYC gate before the next payout.**
  - **Persona + context:** Collections are working. Cumulative successful collections this calendar month exceed 500,000 XOF, or she changes payout destination.
  - **Entry state:** Informal Merchant, no tax ID on file.
  - **Path:** (1) Platform blocks further payouts (not further invoicing). (2) She uploads NINEA/RCCM or national ID. (3) Payouts resume after review.
  - **Climax:** She understands why money is held at the gate — not a silent freeze.
  - **Resolution:** Same account; higher KYC state; CountryPack documents the rule.
  - **Capabilities:** KYC-lite threshold, payout destination change gate. → FR-43, FR-44.

## 3. Glossary

Downstream artifacts use these terms exactly. Synonyms in running prose are a discipline violation.

- **Merchant** — A business that collects recurring money through the platform. One Merchant account may use the Merchant Portal and API keys together.
- **Customer** — The person who consents and pays (or whose cash is marked). Identity key is MSISDN + country code. Not “user.”
- **Client Record** — The Merchant’s roster row for a Customer (name + MSISDN). Created in the portal or implied by a Hosted Checkout Session.
- **Merchant Portal** — The web surface where a Merchant without (or with) an app configures the business, Plans, Client Records, collections, and Developers settings.
- **Hosted Checkout Session** — The API-created session that yields a hosted URL. Completing it runs the Consent State Machine and may collect the current Obligation.
- **Hosted Checkout Page** — The platform-hosted French page the Customer uses for identify → consent → method → confirm. Stripe-like in job, MoMo-shaped in chrome.
- **Consent State Machine** — The single protocol: identify → consent → method → confirm. Rendered on the Hosted Checkout Page and via SMS verbs. Designed so a later USSD renderer can attach.
- **Standing Authorization** — The Customer’s recorded consent to be billed on the Plan’s schedule via reminder-then-pay. A later Direct Debit Port requires a new version; this consent is not a future-mandate.
- **Subscription** — The packaged relationship: Plan + Standing Authorization + the series of Obligations. Not a separate product from send-a-bill.
- **Plan** — Frequency, due dates, amount, late-fee policy, and collection preferences. Separated from which Rail collects.
- **Obligation** — What is owed for a cycle (amount, due date, late fee if applied). Survives a failed Collection Attempt.
- **Collection Attempt** — One try to collect an Obligation (Orange, later other Rails, or Cash Mark-paid). Carries an idempotency token.
- **Collection Primitive** — Invoice → notify → Customer confirms → Rail debit or Cash Mark-paid → Merchant credit. Recurring is this primitive on a schedule.
- **Reminder-then-pay** — The only live collection posture: the platform reminds; the Customer confirms each Collection Attempt. Never merchant-initiated wallet pull.
- **Rail** — A payment method/network. MVP live Rail: Orange Money. Designed Rails: Moov Money, Wave. Future scheme: PI-SPI via Direct Debit Port.
- **Rail Adapter** — The interface that hides the commercial counterparty (local Orange merchant API or BF aggregator). Product requires successful Orange collect and settlement visibility, not a named vendor.
- **Direct Debit Port** — Vacant capability socket for future A2A / PI-SPI debit. Not live in MVP.
- **Cash Mark-paid** — First-class ledger event: the Merchant records that this Obligation was paid in cash. No Take-rate.
- **SMS Invoice** — Transactional SMS stating what, to whom, how often, how much, by when, and how to pay.
- **SMS Receipt** — Transactional SMS proving a successful Collection Attempt or Cash Mark-paid.
- **SMS Verb** — Designed keywords (PO-locked plus exit/resume): PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, REPRENDRE. Matching is accent-insensitive. Surrounding SMS sentences are French.
- **Magic Link** — One-time URL in SMS that identifies the Customer without a password.
- **OTP** — One-time code that identifies the Customer without a password.
- **Staff PIN** — Merchant-side PIN for shared devices.
- **Failed-collection Retry Inbox** — Merchant surface listing failed Collection Attempts with last Rail status and actions: retry or Cash Mark-paid.
- **Pause** — Customer or Merchant temporarily stops generating new Obligations while the Subscription remains.
- **Resume** — End a Pause. Designed SMS verb: REPRENDRE. Also available to the Merchant on the Client Record.
- **Skip** — Omit the current (or next) Obligation without Cancelling the Subscription.
- **Cancel** — Merchant or Customer ends the Subscription: no new Obligations; Standing Authorization revoked; billing SMS stop. Open Obligations remain due until collected, Skipped, or Written-off. Designed SMS verb: ANNULER. Distinct from Session Webhook `cancelled` and from STOP.
- **Write-off** — Merchant closes an open unpaid Obligation as not-to-be-collected. No Take-rate. Not a refund of collected funds.
- **Rail Reversal** — The live Rail reports that a previously `collected` Collection Attempt was reversed. Ledger records it; that Attempt’s Take-rate is reversed; the Obligation re-opens for the reversed amount. Does not reset “first successful MoMo collection ever.”
- **Receivables Book** — Merchant Portal Accueil: who-owes-what (pending consent, open Obligations, recent Collection Attempts).
- **Payout Destination** — Where Pass-through credits the Merchant. MVP: an Orange Money MSISDN.
- **Invite SMS** — Transactional SMS that starts the Consent State Machine for a Client Record.
- **MoMo** — Mobile-money Rails collectively. Live: Orange Money. Designed: Moov Money, Wave. Not a third product.
- **Payout Clock** — Visible timestamps: Customer-paid (immediate on Rail success) and Merchant-received (target T+1 business day), or pending settlement + last Rail status.
- **Pass-through** — Settlement stance: funds are not held as Customer balances. Prefer credit toward the Merchant payout destination. Temporary float, if legally required, is an architecture/legal item — not a Customer wallet product.
- **Take-rate** — Product policy: 2.5% of the successful MoMo collection amount after the Merchant’s first successful MoMo collection ever. Not charged on failed Attempts or Cash Mark-paid. Configurable later; not a forever hardcoded constant in spirit.
- **CountryPack** — Locale pack: strings, Rails, KYC rules, holidays, MSISDN parsing, wallet prefixes, currency. Burkina Faso is the first pack. No `if-Burkina` product logic.
- **MSISDN** — Mobile number in international form; primary Customer (and typically Merchant) identity.
- **Webhook** — Server callback to the Merchant app with Session / Collection Attempt status.
- **Sandbox** — Non-live environment that can complete the Consent State Machine and fake Rail outcomes without live Orange.
- **XOF** — Currency of the Burkina Faso CountryPack (F CFA). Represented as a CountryPack attribute, never a global hardcoded constant in product logic.
- **NINEA** — Informal-to-formal business identifier used at the soft KYC gate (with RCCM or national ID as accepted alternatives).
- **RCCM** — Trade register extract accepted at the soft KYC gate.
- **PI-SPI** — BCEAO interoperable instant-payment scheme. Landscape is moving (sockets / sandbox). Live debit is Out of Scope.

## 4. Features

### 4.1 Merchant workspace (one account, two skins)

**Description:** Informal Merchants start with name, phone, and neighbourhood. The same Merchant account exposes the Merchant Portal and API keys immediately. Developers get a Sandbox even before the first live collect. Shared-device reality is first-class via Staff PIN. Realizes UJ-1, UJ-3, UJ-5.

**Functional Requirements:**

#### FR-1: Informal Merchant signup

A new Merchant can create an account with display name, MSISDN, and neighbourhood, without NINEA, RCCM, or tax ID. Realizes UJ-1.

**Consequences (testable):**
- Signup succeeds when name, MSISDN, and neighbourhood are present and the MSISDN is not already a Merchant.
- Signup is not blocked for missing tax ID or NINEA.
- The Merchant can authenticate on subsequent visits with that MSISDN (OTP or equivalent Merchant-side proof — not a Customer password).

#### FR-2: Sub-15-minute Merchant sitting

A new Merchant can, in one sitting, create a Plan, add the first Client Record, and trigger the first invite SMS in under 15 minutes on a typical 3G connection. Realizes UJ-1.

**Consequences (testable):**
- After signup, the happy path is Plan → Client Record → send invite, with no forced extra modules.
- Time-to-first-invite is a measured success metric (SM-1), not an unmeasured slogan.

#### FR-3: Portal and API on one Merchant account

The same Merchant account can enable the Merchant Portal and API keys in MVP. Realizes UJ-1, UJ-3.

**Consequences (testable):**
- A portal-created Merchant can create API keys without opening a second account.
- An API-created Merchant can open the Merchant Portal and see the same Plans, Client Records, and Obligations.
- Obligations created via Hosted Checkout Session appear in the portal receivables book.

#### FR-4: Staff PIN on shared devices

A Merchant can set a Staff PIN and must enter it to access collection actions and payout settings on a shared device session. Realizes UJ-1, UJ-4.

**Consequences (testable):**
- With Staff PIN enabled, a resumed browser session prompts for the PIN before Cash Mark-paid, retry, payout-destination change, or API-key reveal.
- Staff PIN is optional to set, recommended during first sitting.

#### FR-5: Immediate Developers surface and always-on Sandbox

A Merchant can create Sandbox and live API keys immediately (Developers toggle). Sandbox does not require a prior live collect. Realizes UJ-3.

**Consequences (testable):**
- Sandbox keys work before any successful live collection.
- Live keys may collect only after Orange live credentials/onboarding for that Merchant (or platform-level live Rail) are in place — but the Developers surface itself is not gated on first success.
- Sandbox never initiates a live Orange debit.

### 4.2 Plans, Client Records, and Obligations

**Description:** A Subscription is Plan + Standing Authorization + Obligations. The Plan is what is owed; the Rail is chosen at pay time. The first bill uses the same Collection Primitive as every later cycle (send-a-bill wedge, not a second product). Realizes UJ-1, UJ-2.

**Functional Requirements:**

#### FR-6: Configure a Plan

A Merchant can create and edit a Plan: frequency, due dates, amount (XOF), late-fee policy, and payment collection preferences. Realizes UJ-1.

**Consequences (testable):**
- Supported frequencies in MVP include weekly and monthly. `[ASSUMPTION: term / school-calendar frequencies are NEXT vertical templates; MVP may also offer a simple custom interval of N days.]`
- Due date is visible on the Plan and on every SMS Invoice.
- Late-fee policy can be none, fixed XOF, or percentage of the Obligation; when set, an overdue Obligation includes the computed late fee on the next SMS Invoice.
- Collection preferences do not lock a Rail. Reminder budget is FR-53 (not a free-form Merchant cadence in MVP).

#### FR-7: Register a Client Record by phone

A Merchant can add a Client Record with name + MSISDN in under 60 seconds and trigger an invite SMS. Realizes UJ-1.

**Consequences (testable):**
- Required fields are name and MSISDN only.
- Duplicate MSISDN for the same Merchant is rejected with an explicit message (no silent merge).
- Invite SMS is sent only after the Merchant confirms send.

#### FR-8: Plan separated from Rail

The Customer picks the Rail at the Consent State Machine `method` step (pay time). The Plan does not permanently bind Orange (or a future Rail). Realizes UJ-2. FR-19 is an alias of this rule.

**Consequences (testable):**
- Method step is separate from consent step.
- Standing Authorization remains valid if the Customer pays one cycle Orange and a later cycle (NEXT) on another live Rail.
- MVP live Rail list on the Hosted Checkout Page is Orange Money plus Cash Mark-paid as a Merchant-side action (Customer is not required to “pay cash” in-page).

#### FR-9: First bill is the same primitive

Creating the first Obligation (including a send-a-bill / first cycle) uses invoice + reminder + pay, not a distinct one-off product. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- The first SMS Invoice and the nth cycle SMS Invoice share the same fields and Consent State Machine.
- There is no separate “one-off invoice product” IA in MVP. `[NON-GOAL for MVP: named one-off invoicing product.]`
- Obligation birth rules are FR-53 (not at Invite SMS).

### 4.3 Consent State Machine and Hosted Checkout

**Description:** One machine, two skins. Orange Money PIN and Rail credentials never sit on the Merchant’s servers. Live consent is reminder-then-pay only; a later Direct Debit Port requires a new Standing Authorization version — never bundled into today’s copy. Realizes UJ-2, UJ-3.

**Functional Requirements:**

#### FR-10: Single Consent State Machine

Every Customer enrollment — portal invite or Hosted Checkout Session — runs identify → consent → method → confirm. Realizes UJ-2, UJ-3.

**Consequences (testable):**
- A Customer cannot reach confirm without identify and consent.
- Portal-origin and API-origin sessions persist the same Standing Authorization object type.
- Abandoned sessions leave no collected Obligation and no silent debit.

#### FR-11: Honest reminder-then-pay consent

Consent copy describes scheduled Obligations and reminder-then-pay for as long as the Standing Authorization lasts. The next due date is shown before it happens. Copy never claims merchant-initiated pull, “set and forget,” or that a future debit is already enabled. A later Direct Debit Port requires a **new** Standing Authorization version — today’s consent is not a future-mandate. Realizes UJ-2.

**Consequences (testable):**
- Hosted Checkout Page and SMS consent text include: Merchant name, amount or amount rule, frequency, next due date, that the Customer must confirm each Collection Attempt, and that they can PAUSE, REPRENDRE, or ANNULER (Cancel).
- No UI string in MVP uses “prélèvement automatique,” “auto-debit,” “until automatic debit is enabled,” or equivalent as the live promise.
- Standing Authorization stores the consented text version for audit (NFR-12).
- Enabling a Direct Debit Port in a later version cannot reuse this text version; the Customer must re-consent.

#### FR-12: Customer identity without passwords

A Customer identifies with Magic Link or OTP only. Realizes UJ-2.

**Consequences (testable):**
- There is no Customer password create/reset flow.
- Magic Link and OTP expire after **15 minutes** (`[ASSUMPTION]`); replay after expiry fails.
- Shared-phone risk is disclosed in consent; the enrolled Customer MSISDN is the consenting party.
- Rail-wallet MSISDN may differ from the enrolled MSISDN (whoever confirms on Orange pays for this number). No share-pay-link, second payer, or beneficiary fields in MVP. `[NON-GOAL for MVP: family/group pay product.]`

#### FR-13: Merchant-named trust chrome

Transactional SMS and the Hosted Checkout Page display the Merchant’s shop name (and neighbourhood when space allows) so the Customer can match the real-world provider. Realizes UJ-2.

**Consequences (testable):**
- SMS sender/body includes the Merchant display name used at signup.
- Hosted Checkout Page header shows the same name; mismatch is a defect.

#### FR-14: Create Hosted Checkout Session

A Developer Merchant can create a Hosted Checkout Session that returns a hosted URL for a Customer MSISDN and an existing Plan ID. Realizes UJ-3.

**Consequences (testable):**
- Session create in Sandbox returns a URL without live Orange.
- Session create requires an idempotency key and a Plan ID (no free-form “line items” in MVP).
- Expired Session URL does not collect. `[ASSUMPTION: Session TTL is 24 hours.]`

#### FR-15: Hosted Checkout Page (French MoMo, no card chrome)

The Hosted Checkout Page is French-first, XOF, mobile-web, with no English-default, no `$`, and no card-network logos. Realizes UJ-2, UJ-3.

**Consequences (testable):**
- First-run page language is French.
- Amounts display with F CFA / XOF, never `$`.
- No Visa/Mastercard/Amex marks on the MVP page.

#### FR-16: Webhook and callback on completion

On Session completion or terminal failure, the platform sends a Webhook to the Merchant with status. Realizes UJ-3.

**Consequences (testable):**
- Session statuses include at least: `authorized` (consent recorded, no collect yet), `collected`, `failed`, `expired`, `cancelled`. Session `cancelled` ≠ Subscription Cancel (FR-51).
- Webhook replay does not create a second Collection Attempt debit (FR-24).
- Merchant can rotate a signing secret from the Developers surface.

#### FR-17: Secrets never on Merchant servers

Orange Money PIN, Rail-wallet OTP, and Rail credentials never pass through the Merchant app. Realizes UJ-3.

**Consequences (testable):**
- Hosted Checkout Session API accepts no Orange Money PIN field.
- Merchant-visible APIs expose Session and Collection Attempt status and tokens, not Rail secret material.

#### FR-19: Pay-time method choice

Alias of FR-8: the Customer chooses the Rail at the Consent State Machine `method` step. Keep this ID for UJ citations.

### 4.4 Collection: reminder-then-pay, Orange, cash

**Description:** Live path is reminder-then-pay on Orange Money. Adapters are designed for Moov Money and Wave but those Rails are not live. Cash Mark-paid is first-class. Failures leave the Obligation open. Realizes UJ-2, UJ-4.

**Functional Requirements:**

#### FR-20: Reminder-then-pay only live path

The platform collects live funds only after a Customer confirmation in that Collection Attempt (or Cash Mark-paid by the Merchant). Realizes UJ-2.

**Consequences (testable):**
- No job, API, or portal action can debit Orange without a Customer confirm for that Attempt.
- Scheduled jobs may send SMS Invoice / reminders; they must not debit.

#### FR-21: Successful Orange collect and settlement visibility

The platform can complete a live Orange Money Collection Attempt and show settlement on the Payout Clock. The Rail Adapter hides whether the counterparty is a local Orange merchant API or a BF aggregator. Realizes UJ-2, UJ-3.

**Consequences (testable):**
- A confirm on the Hosted Checkout Page or the no-browser PAY + OTP + Rail-wallet confirm path (FR-54) can reach `collected` on Orange for a configured live Merchant.
- Rail-wallet confirm is operator USSD or in-app — not a platform `*shortcode#`.
- After `collected`, Customer-paid timestamp is set immediately (FR-35).
- Merchant-received moves to target T+1 visibility or `pending settlement` with last Rail status (FR-36, FR-37).
- Product acceptance does not name a required aggregator.

#### FR-22: Designed adapters for Moov Money and Wave

The collection interface allows additional Rails to be attached without changing Merchant Portal concepts or Hosted Checkout Session semantics. Moov Money and Wave are not live in MVP. Realizes the dual-channel vision; `[NON-GOAL for MVP: live Moov or Wave collect.]`

**Consequences (testable):**
- Rail is an enumerated CountryPack field, not a hardcoded single-vendor checkout.
- Enabling a new Rail does not require a new Merchant account type.

#### FR-23: Cash Mark-paid

A Merchant can mark an open Obligation paid in cash. Realizes UJ-1, UJ-4.

**Consequences (testable):**
- Cash Mark-paid closes the Obligation, writes SMS Receipt (unless Customer STOP’d non-essential only — receipt remains transactional and is still sent), and takes no Take-rate.
- Cash Mark-paid requires Staff PIN when enabled (FR-4).
- Cash Mark-paid cannot be applied to an already `collected` Obligation.

#### FR-24: Idempotent Collection Attempts

Retries, double SMS, double confirm, and Webhook replay must not double-charge. Realizes UJ-3, UJ-4.

**Consequences (testable):**
- Two confirms for the same Collection Attempt token result in at most one Orange debit.
- A retried Attempt for the same Obligation uses a new token and is a new Attempt.

#### FR-25: Obligation remains on failure

A failed Collection Attempt does not cancel the Obligation or the Standing Authorization. Realizes UJ-2, UJ-4.

**Consequences (testable):**
- After Orange failure, receivables book still shows amount due.
- Customer and Merchant are notified of failure (FR-29), not of cancellation.

#### FR-26: Failed-collection Retry Inbox

The Merchant can see failed Attempts and retry or Cash Mark-paid. Realizes UJ-4.

**Consequences (testable):**
- Inbox lists Obligation, Customer name/MSISDN, last Rail status, timestamp.
- Retry creates a new Collection Attempt and sends a new reminder/PAY path.

### 4.5 Notifications and SMS verbs

**Description:** SMS is the OS. Platform sends transactional Invite SMS, SMS Invoice, SMS Receipt, and failure messages. Designed verbs: PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, REPRENDRE. STOP ends marketing-like / non-essential messages and is **not** Cancel; transactional billing SMS continue until ANNULER, Merchant Cancel, or Client Record removal — disclosed in consent in 8th-grade French. SMS cost is bundled into the MoMo Take-rate for MVP. `[NOTE FOR PM: cash-only Merchants generate SMS cost at 0% Take-rate — accept subsidy until first Orange collect.]` Realizes UJ-2. Live platform USSD shortcode is NEXT.

**Functional Requirements:**

#### FR-18: Invite SMS

After a Client Record is registered (or a Session needs Customer action), the platform can send an invite SMS with instructions to complete the Consent State Machine. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Invite includes Merchant name, what they are joining, and Magic Link and/or OTP instructions and/or PAY verb.
- Invite is transactional (not stopped by STOP).

#### FR-29: Transactional SMS set

The platform sends SMS for invite, SMS Invoice, SMS Receipt, and collection failure. Realizes UJ-2, UJ-4.

**Consequences (testable):**
- SMS Invoice answers what / to whom / how often / how much / by when / how to pay.
- Failure SMS does not claim the Subscription is cancelled.
- These four remain sendable after STOP (FR-31).

#### FR-30: Designed SMS verbs

The Customer can text PAY, PAUSE, SOLDE, AIDE, STOP, ANNULER, REPRENDRE (accents optional in matching). Surrounding sentences are French. Realizes UJ-2.

**Consequences (testable):**
- PAY starts the current Collection Attempt confirm path (Hosted Checkout Page and/or FR-54 no-browser path).
- PAUSE maps to FR-27; REPRENDRE maps to Resume.
- SOLDE returns current open amount, due date, and Merchant name. SOLDE never shows a platform-held Customer balance.
- AIDE returns the verb list, including that STOP ≠ ANNULER.
- ANNULER maps to FR-51.
- Unknown text receives a short AIDE-equivalent, not silence — except after STOP, AIDE-equivalent is sent only if the Customer originated the text.

#### FR-31: STOP mapping

STOP cancels further non-essential SMS. It does **not** Cancel the Subscription. Transactional billing SMS continue until ANNULER, Merchant Cancel (FR-51), or Client Record removal (FR-52). Mapping is disclosed in consent. Realizes UJ-2.

**Consequences (testable):**
- After STOP: no promo / tip / marketing-like SMS (MVP has none — STOP still records the preference).
- After STOP: SMS Invoice, PAY reminders (capped by FR-53), failure, and SMS Receipt still send. Invite SMS is not re-sent after STOP unless the Merchant creates a new Consent path and the Customer has not ANNULERed.
- Consent text states in French that STOP ≠ ANNULER and that bills will continue until they Cancel.
- A second STOP within 24 hours replies with ANNULER instructions only (no extra invoice).
- After STOP, unsolicited AIDE broadcasts are not sent.

#### FR-32: Silence means success

The platform does not send celebratory extra SMS after success beyond the single SMS Receipt (and Merchant portal update). Realizes UJ-2.

**Consequences (testable):**
- One success does not produce a sequence of “thank you” extras in MVP.
- Failure, Pause, Cancel, and Rail Reversal produce SMS; success produces only the SMS Receipt.

#### FR-33: Named SMS

Invite, invoice, receipt, and failure SMS identify the Merchant by shop name. Realizes UJ-2.

**Consequences (testable):**
- Body includes Merchant display name.
- AIDE and SOLDE also include Merchant display name when they refer to a balance.

### 4.6 Ledger, proof, and Payout Clock

**Description:** Settlement finality is the product. Ledger events are gateway-agnostic. Prefer Pass-through; do not design Customer wallets. Realizes UJ-2, UJ-3, UJ-4.

**Functional Requirements:**

#### FR-34: Gateway-agnostic ledger

Each Collection Attempt records intent, authorized, collected, settled, failed. Write-off of an open Obligation is FR-55. Rail Reversal of a `collected` Attempt is FR-56. `[NON-GOAL for MVP: Merchant-initiated refund of collected funds.]` Realizes UJ-4.

**Consequences (testable):**
- Orange vs Cash Mark-paid share Obligation identity and differ on Collection Attempt method.
- Receivables Book can answer who owes what without reading Rail payloads.
- Ledger has no Customer-visible wallet balance.

#### FR-35: Customer-paid timestamp

On Rail success, Customer-paid is shown immediately. Realizes UJ-2, UJ-4.

**Consequences (testable):**
- Portal and Customer SOLDE/receipt reflect Customer-paid within the success path, not after T+1.

#### FR-36: Merchant-received target T+1

Merchant-received target is T+1 business day on the Payout Clock, published as a target SLA, not a hard guarantee if the Rail delays. Realizes UJ-4.

**Consequences (testable):**
- Clock shows the target date using the Burkina Faso CountryPack business-day calendar. `[ASSUMPTION: holidays come from the CountryPack.]`
- Copy labels it a target, not a guarantee.

#### FR-37: Pending settlement

If Merchant-received is late, the UI shows pending settlement and the last Rail status. Realizes UJ-4.

**Consequences (testable):**
- A collected-but-not-settled Attempt cannot look like “failed.”
- Last Rail status string is visible to the Merchant.

#### FR-38: Pass-through, no Customer wallets

The product does not offer Customer stored balances or wallets. Settlement prefers Pass-through to the Merchant payout destination. Realizes UJ-5 (payout destination is a Merchant attribute).

**Consequences (testable):**
- No Customer-facing “wallet balance” feature exists.
- Merchant registers a Payout Destination (Orange Money MSISDN in MVP). `[ASSUMPTION: Payout Destination is an Orange Money MSISDN.]`
- If legal review requires a temporary platform float, that float is operator-only, is not exposed as a Customer wallet, and must not satisfy SOLDE. Do not implement float until counsel writes the control. `[NOTE FOR PM: legal open item — see §8.]`
- `[NON-GOAL for MVP: platform-visible Customer balance, prepaid envelope, or Merchant-wallet reload.]`

#### FR-39: Visible Payout Clock

Merchant Portal (and Developer-visible payment objects) expose Customer-paid and Merchant-received. Realizes UJ-3, UJ-4.

**Consequences (testable):**
- Both timestamps (or pending) appear on the Obligation detail.

### 4.7 Economics and KYC gates

**Description:** Merchant pays nothing until the first successful MoMo collection ever for that Merchant; then 2.5% Take-rate on successful MoMo amounts. No Take-rate on failed Attempts or Cash Mark-paid. Informal KYC until a soft threshold. Realizes UJ-3, UJ-5.

**Functional Requirements:**

#### FR-40: Take-rate after first success

Take-rate is 2.5% of the Rail-collected XOF on a successful MoMo Collection Attempt, starting after that Merchant’s first successful MoMo collection ever. Realizes UJ-3.

**Consequences (testable):**
- The first successful live Orange collect for a Merchant has 0% platform Take-rate.
- Each later successful live Orange collect charges 2.5% of **Rail-collected XOF** on that Attempt (integer XOF; half-up). If collected XOF = 0, Take-rate = 0.
- Partial Rail success: Take-rate applies to collected XOF only; remainder stays an open Obligation on a new Collection Attempt.
- Late fee is inside the collected amount if that amount was collected.
- Fee is computed at `collected`, not at `settled`. Failed settlement after `collected` does not void the Take-rate unless FR-56 Rail Reversal fires.
- Rail Reversal of Attempt N reverses that Attempt’s Take-rate and does **not** reset “first success ever.”
- Sandbox `collected` never increments the first-success counter and never charges Take-rate.
- Policy is documented as configurable later; MVP implements 2.5% as the sole live value.

#### FR-41: No Take-rate on failed Attempts

Failed Collection Attempts are not charged. Realizes UJ-4.

**Consequences (testable):**
- A failed Orange Attempt adds 0 to platform fees.

#### FR-42: No Take-rate on Cash Mark-paid

Cash Mark-paid is not charged; SMS cost is absorbed in the MoMo Take-rate for MVP. Realizes UJ-1.

**Consequences (testable):**
- Cash Mark-paid fee line is 0.
- There is no separate per-SMS invoice to the Merchant in MVP.

#### FR-43: Soft KYC threshold and payout-destination change

If cumulative successful collections — defined as live MoMo `collected` XOF **plus** Cash Mark-paid XOF — exceed 500,000 XOF in a calendar month, or the Merchant changes Payout Destination, the platform requires NINEA/RCCM or national ID upload before further payouts. Realizes UJ-5.

**Consequences (testable):**
- The FR-43 sum includes Cash Mark-paid (unlike Take-rate “first success,” which is MoMo-only).
- Crossing the threshold does not delete Plans or stop SMS Invoices.
- Further payouts (Merchant-received credits) are blocked until documents are uploaded and accepted.
- Changing Payout Destination while informal triggers the same gate.
- Exact document checklist may be refined in ops without changing this gate. `[ASSUMPTION: “accepted” may be manual review in MVP.]`

#### FR-44: Informal start

Merchants below the gate operate with name + phone + neighbourhood only. Realizes UJ-1.

**Consequences (testable):**
- Payouts may proceed while under threshold and payout destination is unchanged.

### 4.8 Locale, CountryPack, and future sockets

**Description:** Burkina Faso first; CountryPack skeleton from day one; no second-country production; vacant Direct Debit Port; USSD-ready model without a live shortcode. Realizes the long-term vision without diluting MVP.

**Functional Requirements:**

#### FR-45: CountryPack skeleton

Locale, Rails, KYC rules, holidays, MSISDN parsing, wallet prefixes, and currency are represented as a CountryPack. Burkina Faso is the only production pack. Realizes multi-country vision.

**Consequences (testable):**
- Product logic does not branch on a hardcoded “Burkina” special case outside CountryPack data.
- A paper shadow pack for another country may exist in docs only — not a production toggle.

#### FR-46: MSISDN identity

Customer and Client Record identity is MSISDN + country code, not email. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Email is not required for Customer signup or pay.
- MSISDN parsing uses CountryPack rules.

#### FR-47: Currency as CountryPack attribute

Amounts are XOF in the Burkina Faso pack and are not a global hardcoded currency in product rules. Realizes FR-45.

**Consequences (testable):**
- Display and ledger currency resolve from the Merchant’s CountryPack.

#### FR-48: Direct Debit Port socket (vacant)

Live PI-SPI debit and Request-to-Pay are Out of Scope. Architecture may keep a named empty interface. Product FRs do not call it. Realizes future-debit posture without shipping debit.

**Consequences (testable):**
- No Merchant or Customer control starts a PI-SPI debit.
- No Adapter call and no Sandbox toggle posts a PI-SPI debit in MVP.
- FR-11 live consent is not a mandate for this port.

#### FR-49: No live platform USSD shortcode

MVP ships SMS + hosted web only. A future platform USSD renderer is NEXT. Rail-wallet confirm (operator USSD/in-app) is not a platform shortcode. Realizes binding posture.

**Consequences (testable):**
- No MVP FR depends on a live platform `*shortcode#`.
- NFR-8 is satisfied by SMS verbs + Hosted Checkout Page, not by shipping a renderer.

#### FR-50: Sandbox without live Orange

Developers can complete Session + Webhook flows with simulated Rail outcomes. Realizes UJ-3.

**Consequences (testable):**
- Sandbox `collected` does not move live money and does not count as first success (FR-40).
- Sandbox can simulate `failed` for Failed-collection Retry Inbox testing.

### 4.9 Subscription lifecycle

**Description:** Pause, Skip, Cancel, Obligation birth, no-browser confirm, Write-off, and Rail Reversal. Realizes UJ-1, UJ-2, UJ-4.

**Functional Requirements:**

#### FR-27: Basic Pause

Customer (SMS PAUSE) or Merchant can Pause a Subscription. Realizes UJ-2.

**Consequences (testable):**
- While Paused, no new Obligations are generated; open Obligation remains until collected, Skipped, Written-off, or left due.
- Customer Resumes with SMS REPRENDRE; Merchant Resumes on the Client Record.

#### FR-28: Basic Skip

Merchant or Customer can Skip the current or next open Obligation without Cancelling the Subscription. Realizes UJ-2.

**Consequences (testable):**
- Skipped Obligation is closed as skipped, not collected; no Take-rate.
- Following cycle still generates on schedule.

#### FR-51: Cancel Subscription

Customer (SMS ANNULER) or Merchant can Cancel a Subscription. Realizes UJ-2.

**Consequences (testable):**
- After Cancel: no new Obligations; Standing Authorization revoked; billing SMS stop.
- Open Obligations remain due until collected, Skipped, or Written-off.
- Platform emits a Subscription-level cancelled event distinct from Session `cancelled`.
- Merchant Cancel requires Staff PIN when enabled.

#### FR-52: Remove Client Record

A Merchant can remove a Client Record. That Cancels every Subscription with that Customer for this Merchant and stops billing SMS. Realizes UJ-2.

**Consequences (testable):**
- Historical SMS Receipts and ledger rows remain visible to the Merchant.
- Removal requires Staff PIN when enabled.
- Customer SOLDE after removal returns that this Merchant has no open Subscription (not a wallet balance).

#### FR-53: Obligation birth and reminder budget

An Obligation is minted when Standing Authorization is confirmed, for the next due date on the Plan. If that due date is today or in the past, the Obligation is due immediately. Before consent, the Receivables Book shows pending consent — not an owed amount. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Invite SMS does not create an Obligation.
- Abandoned Hosted Checkout Session creates no Obligation if consent never completed.
- A Hosted Checkout Session that collects now may mint and collect the current Obligation in the same Session.
- While Paused, no new Obligations mint (FR-27). Without Standing Authorization, no Obligation mints.
- At mint if already due, and on each later due date: **one** SMS Invoice.
- If the Obligation is still open **48 hours** after the SMS Invoice (`[ASSUMPTION]`), the platform may send **at most one** reminder SMS. No further PAY pings for that Obligation unless the Merchant retries (FR-26).
- Subsequent cycles mint on the Plan schedule after each due date while the Subscription is active and not Paused.

#### FR-54: No-browser Collection Attempt confirm

A Customer on a feature phone can complete a Collection Attempt without opening the Hosted Checkout Page: PAY → OTP (SMS) → Rail-wallet confirm (operator USSD or in-app). Realizes UJ-2.

**Consequences (testable):**
- This path can reach Orange `collected` (FR-21) without a browser.
- It is not a platform USSD shortcode (FR-49).
- Hosted Checkout Page remains an equal path for 3G browsers.

#### FR-55: Merchant Write-off

A Merchant can Write-off an open unpaid Obligation. Realizes UJ-4.

**Consequences (testable):**
- Write-off requires Staff PIN when enabled; Customer cannot Write-off.
- Write-off closes the Obligation; no Take-rate; Customer gets a transactional notice.
- Write-off cannot apply to a `collected` Attempt (use FR-56 if the Rail reversed).

#### FR-56: Rail Reversal

If the live Rail reports a previously `collected` Collection Attempt reversed, the platform records a Rail Reversal. Realizes UJ-4.

**Consequences (testable):**
- That Attempt’s Take-rate is reversed; “first success ever” is not reset.
- The Obligation re-opens for the reversed XOF.
- Customer and Merchant are notified; Payout Clock does not show Customer-paid for the reversed Attempt.

## 5. Non-Goals (Explicit)

- Not a payment institution or e-money issuer this version.
- Not a Customer wallet or Super-App.
- Not auto-debit marketing or a merchant-initiated wallet pull.
- Not live PI-SPI debit or Request-to-Pay (empty interface only; no Adapter call).
- Not Merchant-initiated refund of collected funds (Rail Reversal is in-scope).
- Not a platform-visible Customer balance, prepaid envelope, or Merchant-wallet reload.
- Not live Moov Money or Wave collection (adapters designed only).
- Not a second-country production CountryPack.
- Not native Merchant or Customer apps.
- Not card PAN checkout, English-default, or `$` chrome.
- Not a named one-off invoicing product, vertical template marketplace, or consumer home of all subscriptions.
- Not family/group pay, collector cash-day, WhatsApp/IVR mesh, or live USSD shortcode.
- Not demand-side “Customer invites Merchant” growth.
- Not tontine, credit scoring, B2B supplier credit, NGO/government payers, split/franchise payouts, salary-like outbound, or savings envelopes.

## 6. MVP Scope

### 6.1 In Scope

- Merchant workspace — FR-1–FR-5
- Plans, Client Records, first-bill primitive — FR-6–FR-9
- Consent State Machine and Hosted Checkout — FR-10–FR-17, FR-19
- Orange collect, cash, failure — FR-20–FR-26
- SMS OS and verbs — FR-18, FR-29–FR-33
- Ledger and Payout Clock — FR-34–FR-39
- Take-rate and KYC — FR-40–FR-44
- CountryPack, vacant port, Sandbox — FR-45–FR-50
- Subscription lifecycle (Pause/Skip/Cancel, schedule, no-browser, Write-off, Rail Reversal) — FR-27, FR-28, FR-51–FR-56

### 6.2 Out of Scope for MVP

- Live Moov / Wave / in-session failover — Orange must be trusted first.
- Live USSD shortcode — model ready; cost/availability NEXT.
- WhatsApp receipts, IVR, escalating payday dunning — learn from basic retry first.
- Collector mode, family/group pay, consumer subscription home — trust/UX and demand-side later.
- Named one-off product, vertical templates, PWA/QR/paper card — clone-a-plan + mobile web first.
- Merchant payout-to-own-code productization beyond Pass-through destination — legal/settlement design.
- MRR/churn analytics suite, Laravel/Flutter samples, dispute freeze, Mooré/Dioula keywords.
- Thermal receipt, MNO daily-limit display, known-customer whitelist.
- Live PI-SPI debit, second-country production, cards, native apps, platform Customer balances, payment-institution license, auto-debit copy.
- Full NEXT/LATER lists in `addendum.md` — do not promote them into FRs.

## 7. Success Metrics

**Primary**

- **SM-1**: Time-to-first-invite — median new Merchant sitting from account create → first invite SMS **< 15 minutes**. Validates FR-2, FR-6, FR-7, FR-18.
- **SM-2**: Client Record speed — median add Client Record (name + MSISDN + confirm send) **< 60 seconds**. Validates FR-7.
- **SM-3**: First successful cycle — within 30 days of first Invite SMS, the Merchant records at least one `collected` Orange Attempt **or** Cash Mark-paid. Validates FR-23, FR-53. Does **not** by itself prove the Orange Rail path (see SM-10).
- **SM-4**: Zero double-charge incidents in production (target: **0** duplicate Orange debits per idempotency token). Validates FR-24.
- **SM-5**: Developer time-to-first-Sandbox-Webhook — median **< 1 week** of calendar time from key create (self-reported / measured in onboarding). Validates FR-5, FR-14, FR-16, FR-50.

**Secondary**

- **SM-6**: Payout Clock honesty — share of successful Orange collects where Customer-paid is visible immediately **≥ 99%**. Validates FR-35, FR-39.
- **SM-7**: Settlement visibility — share of collected Attempts that show either Merchant-received or pending settlement + last Rail status within 24 hours **= 100%**. Validates FR-36, FR-37.
- **SM-8**: French-first integrity — first-run Hosted Checkout Page and SMS templates with **0** English-default or `$` / card-logo defects in QA. Validates FR-15, NFR-1.
- **SM-9**: Take-rate correctness — 0% on first live MoMo success and on cash/fail/Sandbox; 2.5% of Rail-collected XOF thereafter; reversal claws that Attempt’s fee only. Validates FR-40, FR-41, FR-42, FR-56.
- **SM-10**: Launch gate — at least one live Orange `collected` in production (not Sandbox, not cash) before calling the Orange path done. Validates FR-21, FR-54.

**Counter-metrics (do not optimize)**

- **SM-C1**: Do not maximize SMS volume per Obligation (spams Fatou; burns the Take-rate bundle). Counterbalances SM-3.
- **SM-C2**: Do not maximize Take-rate collected in month one (would fight “pay nothing until first success” and informal onboarding). Counterbalances SM-9.
- **SM-C3**: Do not maximize Pause/STOP rates as a “engagement” vanity; they are trust valves. Counterbalances SM-3.
- **SM-C4**: Do not treat Cash Mark-paid share as failure — cash is the on-ramp. Counterbalances a naive “% Orange” North Star.

## 8. Open Questions

No remaining **known** phase-blockers among locked product-owner decisions. Recurring engine holes from review (Cancel, Obligation birth, no-browser confirm, Take-rate math) are now FRs. Remaining items are owned and deferred.

1. **Legal float.** If Pass-through is not viable, what temporary float is required? Owner: founder + counsel. Revisit: before live settlement at volume. Do not add Customer wallets; do not implement float until counsel writes the control. `[NOTE FOR PM]`
2. **Exact KYC document checklist and review SLA.** Gate is locked; ops list can refine (NINEA vs RCCM vs CNIB sides, selfie or not). Owner: ops. Revisit: first Merchant approaching 500,000 XOF/month.
3. **Orange commercial counterparty.** Aggregator vs local Orange merchant API. Owner: architecture. Product only requires FR-21.
4. **Late-fee compounding.** Fixed vs percent is in; stacking after multiple overdue cycles needs UX copy. Owner: UX. Revisit: during UX spec.
5. **Webhook delivery SLA and retry schedule.** Capability is in; numeric retry budget is architecture. Owner: architecture.
6. **Merchant authentication factor** beyond MSISDN OTP (email backup). Owner: UX. Not required for Customer.
7. **Cash-only SMS subsidy.** Cash Mark-paid takes 0% while SMS is bundled. `[NOTE FOR PM]` Accept subsidy until first Orange collect; no per-SMS invoice in MVP. Revisit if cash-only books dominate. Owner: founder.

## 9. Assumptions Index

- `[ASSUMPTION]` FR-6: term/school frequencies are NEXT; MVP weekly + monthly + optional N-day interval.
- `[ASSUMPTION]` FR-12: Magic Link and OTP TTL = 15 minutes.
- `[ASSUMPTION]` FR-14: Hosted Checkout Session TTL = 24 hours.
- `[ASSUMPTION]` FR-36 / NFR-5: T+1 uses CountryPack BF holidays.
- `[ASSUMPTION]` FR-38: Payout Destination in MVP is an Orange Money MSISDN.
- `[ASSUMPTION]` FR-43: document acceptance may be manual review in MVP.
- `[ASSUMPTION]` FR-53: at most one reminder, 48 hours after SMS Invoice.
- `[ASSUMPTION]` System-idea “Customer logs in” = Magic Link or OTP (FR-12), not password accounts.
- `[ASSUMPTION]` Take-rate “first successful collection ever” = first successful **live MoMo/Orange** collect, not Cash Mark-paid or Sandbox (FR-40).
- `[ASSUMPTION]` Late-fee policy is applied to overdue Obligations in MVP (FR-6); compounding edges refine in UX.
- `[ASSUMPTION]` NFR-2: Hosted Checkout Page first contentful interaction targets < 8 seconds on a mid-3G QA profile.
- `[ASSUMPTION]` §17: MVP hosting may be outside BF if legal allows; CountryPack must not assume in-country compute.

## 10. Monetization

See FR-40–FR-42. Policy: 0% until first live MoMo success, then 2.5% of Rail-collected XOF; 0% on fail and Cash Mark-paid; SMS bundled; configurable later. Differentiation is the OS, not cheapest checkout (aggregators often cited ~3.5–4.5%).

## 11. Cross-Cutting NFRs

#### NFR-1: French-first locale

All first-run Merchant Portal, Hosted Checkout Page, SMS, and Merchant-facing API errors are French. English may exist in developer technical identifiers, not on the Hosted Checkout Page or in SMS.

#### NFR-2: Feature-phone and 3G viability

The Customer happy path is completable from SMS verbs plus a low-weight Hosted Checkout Page. `[ASSUMPTION: Hosted Checkout Page first contentful interaction targets < 8 seconds on a mid-3G profile in QA.]`

#### NFR-3: Idempotency and exactly-once debit

See FR-24. Webhook at-least-once delivery is allowed; debit is at-most-once per Collection Attempt token.

#### NFR-4: Rail status observability

Every live Attempt stores last Rail status sufficient for FR-37. Internal operators can see Adapter-level outcome without exposing secrets to Merchants.

#### NFR-5: Payout Clock SLA (target)

Customer-paid: immediate on Rail success. Merchant-received: T+1 business day **target**. Delays surface pending settlement. Not a hard guarantee against Rail outages.

#### NFR-6: Secret isolation

FR-17. Live API keys are revealed at create and cannot be recovered later in full. Staff PIN cannot be recovered in plaintext. Storage mechanics are architecture.

#### NFR-7: MSISDN and PII minimization

MSISDN is the identity key. No Customer email harvest in MVP. Retention of SMS bodies and consent text is sufficient for dispute/audit (NFR-12) and is documented in a privacy notice (French).

#### NFR-8: Literacy-first verbs

Numeric/SMS verbs are not a degraded mode. Hosted web is equal, not superior. NFR-8 is tested on SMS + Hosted Checkout Page, not on a future platform USSD renderer.

#### NFR-9: Multi-country sockets without second production pack

CountryPack + Rail Adapter + vacant Direct Debit Port. No production CI/other pack. Currency and MSISDN rules are data.

#### NFR-10: Webhook reliability

Signed Webhooks; Merchant can retry from the Developers surface; platform retries failed deliveries. Numeric schedule is architecture.

#### NFR-11: Shame-free tone

Dunning and failure copy is private and factual. No public defaulter wall, no shaming ranking in MVP.

#### NFR-12: Consent and collection audit trail

Standing Authorization text version, timestamps, Collection Attempt tokens, Cash Mark-paid actor, and payout-destination changes are retained for audit.

## 12. Constraints and Guardrails

**Safety.** Never debit without Customer confirm (FR-20). Never double-charge (FR-24). STOP cannot silently strand an owed Obligation without transactional notice (FR-31).

**Privacy.** Shared-phone disclosure. Staff PIN. No Customer wallet dossier beyond receivables need.

**Cost.** SMS bundled; do not build chatty notification meshes (SM-C1). Take-rate after first success only.

**License.** Do not hold Customer balances as a product. Do not claim e-money issuance. Operate as software + collection via licensed Rail / aggregator as architecture chooses.

## 13. Compliance and Regulatory

- **WAEMU / BCEAO:** Do not issue e-money; do not market as a payment institution. Prefer agent / technical integrator posture via the Orange Adapter’s licensed counterparty.
- **Consent:** Instruction 001-01-2024-style explicit payer consent — our live model is payer-approved each Attempt.
- **KYC:** Informal start is a product decision; e-money wallet KYC ceilings belong to the Rail, not our invented tiers. Our gate (FR-43) is a platform payout-risk control, not a claim of EME compliance.
- **ARCEP / SMS:** Transactional SMS vs SVA marketing. STOP mapping (FR-31) is the product expression of subscriber-initiative norms. Live shortcode/USSD declaration is NEXT.
- **PI-SPI:** Empty Direct Debit Port only. Do not implement Request-to-Pay or live mandate debit in MVP. BF entity first; per-country entity on expand.
- **Copy law:** Honest reminder-then-pay. Auto-debit claims are a compliance and trust defect.

## 14. Operational Requirements

- Publish Payout Clock targets (NFR-5) in Merchant-facing help.
- Surface last Rail status when settlement is late (FR-37).
- Manual KYC review is acceptable in MVP (FR-43 assumption).
- Support: Merchant Portal + AIDE SMS; no voice support line as a product.
- Orange or aggregator outage: reminders may send; collects fail closed; Obligations remain.

## 15. Integration and Dependencies

- **Orange Money collect** via Rail Adapter (counterparty chosen in architecture).
- **SMS sender** (in-country SMS API or aggregator) — product requires deliverability of FR-29, not a named vendor.
- **Merchant apps** integrate Hosted Checkout Session + Webhook only.
- **PI-SPI** is a future participant-side dependency; not an MVP runtime dependency.
- Moov Money and Wave: Adapter interfaces only.

## 16. API Contracts (capability)

Public surface (names illustrative; exact paths are architecture):

- Create / expire Hosted Checkout Session (idempotency key required).
- Retrieve Session and Obligation / Collection Attempt status.
- Webhook events for terminal and settlement status changes.
- Sandbox counterparts of the above.

**Breaking-change policy (capability):** Merchant-visible status names (`authorized`, `collected`, `failed`, `expired`, `cancelled`, `settled`) are stable. Adding a Rail or Direct Debit Port must not require a new Session type.

Versioning details → `addendum.md`.

## 17. Data Governance

- Identity: MSISDN + country code.
- Residency: `[ASSUMPTION: MVP hosting may be outside BF if legal allows; CountryPack must not assume in-country compute. Revisit with counsel.]`
- Classification: MSISDN, consent text, and ledger are sensitive.
- Retention: keep consent + collection audit for the life of the Merchant relationship plus a documented minimum (ops/legal).
- No sale of Customer ledgers.

## 18. Risk and Mitigations

| Risk | Mitigation |
| --- | --- |
| Marketed as auto-debit | FR-11, FR-20, NFR-11; review all copy |
| Orange BF not on public WebPay lists | Adapter hides counterparty; FR-21 is outcome-based |
| Holding funds → license | FR-38; legal open question §8.1 |
| Informal KYC abuse | FR-43 gate; payout destination change gate |
| SMS STOP vs billing | FR-31 disclosed in consent |
| Double SMS / double pay | FR-24 |
| Shared phones | Staff PIN; Magic Link expiry; shop-name match |
| PI-SPI hype | FR-48 vacant; live debit Out of Scope |
| Feature sprawl vs trust | Scope honesty §5–6; settlement finality over templates |

## 19. Aesthetic and Tone

- French, official-enough receipt, named SMS, neighbourhood-level trust — not neobank English, not card-checkout skeuomorphism.
- Silence after success; ping on failure or change.
- Shame-free private reminder. Cash is an on-ramp, not an apology.
- Developer docs should feel like Stripe, not a bank PDF — French examples in XOF.

## 20. Information Architecture (surfaces, not screens)

| Surface | Actor | Job |
| --- | --- | --- |
| Merchant Portal (mobile web) | Merchant | Onboard, Plans, Client Records, Receivables Book, retry, cash, KYC, Developers |
| Hosted Checkout Page (mobile web) | Customer | Consent State Machine + Orange confirm |
| SMS | Customer | Invite, invoice, receipt, failure, verbs |
| Public API + Webhooks | Developer Merchant | Hosted Checkout Session |
| Sandbox | Developer Merchant | Fake Rail outcomes |

No native apps. Platform USSD is NEXT, not shipped. PWA / shop QR / paper card are NEXT.

**First sitting (Merchant Portal):** Signup (FR-1) → Plan (FR-6) → Payout Destination (FR-38) → Client Record + Invite SMS (FR-7, FR-18). Staff PIN prompt offered before leaving the machine (FR-4).

**Merchant Portal top-level:** Accueil (Receivables Book: pending consent, open Obligations, last Collection Attempt, Payout Clock summary) · Clients (Client Record detail: Pause / Resume / Skip / Cancel / remove) · Plans · Encaissements (Failed-collection Retry Inbox, Cash Mark-paid, Write-off, Payout Clock) · Paramètres (Payout Destination, Staff PIN, KYC upload + hold banner) · Développeurs (keys, Webhooks, Sandbox).

**Action homes:** Cash Mark-paid and retry live on Encaissements and on the open Obligation row. Pause / Resume / Skip / Cancel / remove live on Client Record detail (Staff PIN when enabled). KYC hold is a banner on Accueil **and** Paramètres.

**Hosted Checkout Page:** linear four states = identify → consent → method → confirm (glossary Consent State Machine). Customer has no password account home in MVP (NEXT: consumer home). SMS is a verb list, not a screen.

**French nav labels** (UX may refine): Accueil · Clients · Plans · Encaissements · Paramètres · Développeurs. Glossary names stay English for downstream IDs.
