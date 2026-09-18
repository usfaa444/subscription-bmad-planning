---
name: Sarati
status: final
sources:
  - "{planning_artifacts}/prds/prd-bf-recurring-payments-2026-09-17/prd.md"
  - "{planning_artifacts}/prds/prd-bf-recurring-payments-2026-09-17/addendum.md"
  - "{planning_artifacts}/architecture/architecture-bf-recurring-payments-2026-09-17/ARCHITECTURE-SPINE.md"
  - "{planning_artifacts}/briefs/brief-bf-recurring-payments-2026-09-17/brief.md"
updated: 2026-09-18
---

# Sarati — Experience Spine

> Behavioral contract for the dual-channel receivables-and-consent OS. Paired with `DESIGN.md`. Glossary terms from the PRD are used verbatim. Never say "user" — say **Customer** or **Merchant**. Spines win on conflict with any mock.

## Foundation

Multi-surface, no native apps:

| Surface | App / channel | Form-factor | UI system |
|---|---|---|---|
| **Merchant Portal** | `apps/portal` | Responsive mobile-web first (shared Android + cybercafé PC) | shadcn/ui + Tailwind on Next.js |
| **Hosted Checkout Page** | `apps/checkout` | Phone-column mobile-web (also the Magic Link landing) | same |
| **SMS** | inbound/outbound verbs | Feature phone and 3G Android | transcript patterns, not screens |
| **Developers** | Portal tab + public API | Desktop-capable Portal | same Portal system |
| **Sandbox** | same Portal + Hosted Checkout | Hosted Checkout in `livemode=false` | same Checkout system |

`DESIGN.md` is the visual identity reference. This spine is the experience. Both skins and SMS render **one Consent State Machine**: identify → consent → method → confirm. SMS is not a second protocol.

Locale, currency display, MSISDN parse, and holidays come from the BF **CountryPack**. French-first everywhere a Merchant or Customer can read. English only in developer technical identifiers (`code`, route names), never on checkout or SMS.

**[ASSUMPTION]** Light-only. Product brand is **Sarati** (sarati.net). Trust chrome on Customer surfaces remains Merchant name + neighbourhood; Sarati wordmark may appear as platform chrome.

## Information Architecture

### Merchant Portal

First sitting (forced, no extra modules): **Signup → Plan → Payout Destination → Client Record + Invite SMS → Staff PIN offered**.

| Surface | Reached from | Purpose |
|---|---|---|
| Signup | Unauthenticated | Name, MSISDN, neighbourhood. MSISDN + OTP. No tax ID. |
| Plan create | First sitting / Plans | Amount, frequency (weekly/monthly), due weekday or day, late-fee policy, collection preference. |
| Payout Destination | First sitting / Paramètres | Orange MSISDN. Staff PIN when enabled. |
| Client Record create | First sitting / Clients | Name + MSISDN only. Duplicate MSISDN = explicit reject. Invite after confirm send. |
| Staff PIN setup | End of first sitting / Paramètres | Optional, recommended. |
| **Accueil** | App open | **Receivables Book**: pending consent, open Obligations, last Collection Attempt, Payout Clock summary. KYC hold banner if gated. |
| Clients | Tab | Client Record list. |
| Client Record detail | Clients row | Pause / Resume / Skip / Cancel / remove. Staff PIN when enabled. |
| Plans | Tab | Plan list + create. |
| Encaissements | Tab | Failed-collection Retry Inbox, Cash Mark-paid, Write-off, Payout Clock detail. |
| Paramètres | Tab | Payout Destination, Staff PIN, KYC upload + hold banner. |
| Développeurs | Tab | API keys (shown once), Webhooks, Sandbox, signing-secret rotation, replay. |

French nav (PRD §20, locked): **Accueil · Clients · Plans · Encaissements · Paramètres · Développeurs**.

**[ASSUMPTION]** On `< md`, **Développeurs** is a row inside **Paramètres** (five-tab bar). On `lg+`, Développeurs is a sixth top-nav item. Adama still reaches keys without a sixth thumb target on a shared Android.

Action homes: Cash Mark-paid + retry on **Encaissements** and on an open Obligation row. Lifecycle verbs on **Client Record detail**. KYC hold banner on **Accueil and Paramètres**.

### Hosted Checkout Page

Linear four steps. **No Customer account home** in MVP. Session TTL 24h. Magic Link / OTP TTL 15 min.

| Step | Purpose |
|---|---|
| identify | Magic Link land or OTP entry. No password. Shared-phone risk in consent (next step). |
| consent | Standing Authorization copy. Cannot skip. |
| method | Orange Money (live). Customer chooses at pay time. |
| confirm | Hand off to Rail-wallet confirm (operator USSD / in-app). Then SMS Receipt. |

Expired Session URL does not collect. Abandoned Session mints no Obligation.

### SMS (verb list, not a screen)

Invite SMS · SMS Invoice · PAY reminder (≤1, 48h) · failure · SMS Receipt · AIDE · SOLDE · STOP ack · ANNULER ack. After STOP: transactional billing SMS continue; second STOP in 24h → ANNULER instructions only.

### Surfaces that do not exist in MVP

Customer password home · public defaulter wall · platform USSD shortcode · native apps · live Moov/Wave method chips · consumer Super-App of all Subscriptions · second-payer fields.

→ Composition reference: `mockups/portal-first-sitting.html`, `mockups/receivables-book.html`, `mockups/hosted-checkout.html`, `mockups/sms-transcript.html`. Spine wins on conflict.

| Surface | Visual reference |
|---|---|
| First sitting Client Record + Staff PIN | mocked — `mockups/portal-first-sitting.html` |
| Accueil Receivables Book + Payout Clock + KYC banner | mocked — `mockups/receivables-book.html` |
| Checkout identify + consent | mocked — `mockups/hosted-checkout.html` |
| SMS Invoice / PAY / Receipt + AIDE / STOP | mocked — `mockups/sms-transcript.html` |
| Signup, Plan, Payout Destination, Clients list, Client Record detail, Plans, Encaissements full inbox, Paramètres, Développeurs, Checkout method/confirm | spine-only |

## Voice and Tone

Microcopy. Brand posture lives in `DESIGN.md`. Language: French, 8th-grade, shame-free, regulated-honest. **Silence = success.**

| Do | Don't |
|---|---|
| "2 000 F CFA — vendredi 18 sept." | "Your payment of $20 is due!" |
| "En attente de consentement" | "Impayé" before Standing Authorization |
| "Marquer payé (espèces)" | "Paiement manuel (non recommandé)" |
| "Client payé" / "Vous recevez — cible T+1, pas une garantie" | "Règlement instantané garanti" |
| "STOP arrête les messages facultatifs. Pour arrêter le plan : ANNULER." | "STOP to cancel subscription" |
| "Orange n'a pas confirmé. L'échéance reste due." | "Fatou a refusé de payer." |
| "Paiements retenus — les factures continuent." | "Compte bloqué." |
| One SMS Receipt. Then quiet. | "🎉 Paiement réussi !!!" + extra pings |
| "Rappel puis paiement. Vous confirmez chaque fois." | "Prélèvement automatique", "auto-debit", "set and forget" |

Cash is first-class, not an apology. Failure is factual and private. Never name a Customer as a defaulter on any Merchant-visible ranking.

**[ASSUMPTION]** Canonical SMS bodies below are field-complete drafts for mocks and stories. Ops/legal may tighten character counts.

### Canonical SMS bodies (draft)

**Invite SMS**

```
Awa — Nettoyage, Ouaga
Paiement hebdomadaire 2 000 F CFA, chaque vendredi.
Répondez PAY ou ouvrez le lien.
STOP n'annule pas. Pour arrêter : ANNULER
```

**SMS Invoice** (what / to whom / how often / how much / by when / how to pay)

```
Awa — Nettoyage
Nettoyage hebdomadaire — 2 000 F CFA
Échéance : vendredi 18 sept.
Répondez PAY pour confirmer
```

**Failure**

```
Awa — Nettoyage
Orange n'a pas confirmé 2 000 F CFA.
L'échéance du 18 sept. reste due.
Répondez PAY pour réessayer
```

**SMS Receipt**

```
Reçu — Awa — Nettoyage
2 000 F CFA reçus le 17 sept.
Prochaine échéance : vendredi 25 sept.
```

**AIDE**

```
PAY payer · SOLDE voir l'échéance · PAUSE / REPRENDRE
ANNULER arrêter le plan
STOP arrêter les messages facultatifs (les factures continuent)
STOP n'est pas ANNULER
```

**SOLDE** — open amount, due date, Merchant name. Never a platform Customer balance.

## Consent State Machine

One machine, two skins + SMS. CheckoutSession statuses (session only): `authorized` · `collected` · `failed` · `expired` · `cancelled` · `settled`. Do not reuse session words on Attempts except `collected` / `failed` / `settled` with ledger meaning.

| Step | Portal / Invite origin | Hosted Checkout Page | SMS / no-browser |
|---|---|---|---|
| identify | Invite SMS Magic Link or OTP | Magic Link land or OTP field | PAY starts Attempt; OTP over SMS |
| consent | Standing Authorization recorded when Customer confirms on page or via designed SMS confirm | Consent card + explicit confirm | Consent copy in Invite; confirm is the PAY path after identify |
| method | Customer chooses at pay time (not locked at enroll) | Orange Money control | Implicit Orange on PAY; Cash is Merchant-side |
| confirm | Rail-wallet (operator USSD / in-app) or Merchant Cash Mark-paid | Hand-off copy, then wait | OTP + Rail-wallet confirm. Platform has no `*shortcode#` |

Hard rules:

- Cannot reach **confirm** without identify + consent.
- Reminder-then-pay only. Scheduled jobs send SMS Invoice / reminders; they must not debit.
- Standing Authorization is **not** a future mandate. Direct Debit Port needs a new version — never implied by current copy.
- Consent must include: Merchant name; amount or amount rule; frequency; next due date; Customer confirms each Collection Attempt; PAUSE / REPRENDRE / ANNULER; STOP ≠ Cancel; shared-phone risk.
- Store consented text version on Standing Authorization.
- Abandoned Invite / Session: Receivables Book shows **pending consent**, no owed Obligation.
- Obligation mints when Standing Authorization is confirmed for the next due date.
- STOP ends non-essential SMS only. Transactional SMS Invoice, PAY reminders (budget), failure, SMS Receipt continue until ANNULER, Merchant Cancel, or Client Record removal.
- Cancel (ANNULER) ends Subscription; no new Obligations; Standing Authorization revoked; billing SMS stop; open Obligations remain due until collected, Skipped, or Written-off.
- Session `cancelled` ≠ Subscription Cancel.
- Whoever confirms on Orange pays for the enrolled MSISDN. No second-payer fields.

## Dual-skin + SMS mapping

| Need | Portal | Checkout | SMS |
|---|---|---|---|
| Enroll first Client | First sitting | — | Invite SMS |
| See who owes what | Accueil Receivables Book | — | SOLDE (Customer-side only) |
| Pay this cycle | — | method + confirm | PAY + OTP + Rail-wallet |
| Cash received in hand | Cash Mark-paid (Staff PIN) | — | SMS Receipt (same as MoMo) |
| Failed Friday | Retry Inbox | Retry Session if still open | Failure SMS + PAY |
| Pause / skip / cancel | Client Record detail | — | PAUSE / REPRENDRE / ANNULER |
| Quiet extras | — | — | STOP |
| Developer first Session | Développeurs | Sandbox Checkout | Sandbox does not SMS-live Orange |
| KYC before payout | Banner Accueil + Paramètres | — | — |

## Voice-adjacent regulated language

Banned on every surface: prélèvement automatique · auto-debit · set and forget · card-network marks · English-default checkout · `$` · public ranking of late Customers · "compte bloqué" for KYC · SOLDE as wallet balance · platform USSD shortcode as the pay path.

## Component Patterns

Behavioral. Visual specs live in `DESIGN.md.Components` (or shadcn defaults).

| Component | Surfaces | Behavioral rules |
|---|---|---|
| Primary button | Portal, Checkout | One per screen. French verb. Disabled until the required field set is valid. Min `{spacing.tap}`. |
| Status pill | Accueil, Clients, Encaissements | Labels: En attente de consentement · Dû · En retard · Payé · Échec · En pause · Annulé · Radié. Never color-only. |
| Receivables row | Accueil | Tap → Obligation or Client Record detail. Shows pending vs owed honestly. Clay action: Marquer payé on open/failed. |
| Payout Clock | Accueil summary, Obligation detail | Two facts: Client payé (Rail success or Cash Mark-paid) vs Vous recevez (cible T+1 or pending settlement + last Rail status). Reversal clears Client payé. |
| Staff PIN pad | Overlay on guarded actions | Required when set: Cash Mark-paid, retry, payout change, API-key reveal, Cancel, Client Record remove, Write-off. 3 failures → wait copy, no lockout theater. |
| Trust chrome | Checkout header, SMS first line | Merchant display name + neighbourhood when space. |
| Consent card | Checkout consent | All FR-11 fields visible without a "more" accordion on a phone. Confirm is an explicit control. Back does not collect. |
| OTP / Magic Link | Checkout identify, SMS PAY | 15 min TTL. Resend once per minute. No password create. |
| Method picker | Checkout method | Orange Money only in production. **[ASSUMPTION]** Moov/Wave hidden, not disabled-tease. |
| Confirm hand-off | Checkout confirm | Instructs Rail-wallet confirm. Spinner + "Confirmez sur Orange Money." Timeout → Échec, Obligation remains. |
| Cash Mark-paid sheet | Accueil row, Encaissements | Amount (defaults to open Obligation), optional note, Staff PIN. Sends SMS Receipt. Sets Client payé; Vous recevez = déjà reçu. |
| Retry inbox row | Encaissements | New Collection Attempt + reminder/PAY path. Idempotent: never implies a second debit of a succeeded Attempt. |
| KYC hold banner | Accueil, Paramètres | Payouts held; invoicing continues. CTA → upload NINEA/RCCM or national ID. |
| SMS verb chip | SMS transcript mocks, AIDE | Exact verbs, accent-insensitive match. Surrounding sentence French. |
| Empty state | Accueil, lists | One sentence + one primary action. Accueil empty after signup: "Invitez un client — rien n'est dû avant son accord." |
| Developers key row | Développeurs | Live secret shown once. Sandbox keys always available. Replay must not re-debit. |

## State Patterns

| State | Surface | Treatment |
|---|---|---|
| Cold load | Portal, Checkout | shadcn `Skeleton` matching the column. No English placeholder copy. |
| First sitting incomplete | Portal | Resume the next missing step (Plan / Payout Destination / Client Record). Do not open Accueil as if enrolled. |
| Accueil empty | Accueil | Pending-none, owed-none. CTA: Ajouter un client. |
| Pending consent | Accueil row | Status pill En attente de consentement. Amount not shown as owed. No Cash Mark-paid. |
| Open Obligation | Accueil, Encaissements | Dû + due date. Cash Mark-paid and (if failed) Réessayer. |
| En retard | Accueil | Late pill + next SMS Invoice includes late fee when policy set. No shame ranking. |
| Échec (Rail) | Accueil, Retry Inbox, SMS | Obligation remains. Failure SMS. Last Rail status visible to Merchant. |
| Payé / collected | Accueil, Checkout end, SMS | One receipt. Portal row quiet. Checkout may close. No extra pings. |
| Pending settlement | Payout Clock | Client payé yes; Vous recevez = en attente de règlement + last Rail status. Not Échec. |
| Rail Reversal | Payout Clock, SMS | Notify both. Client payé cleared for that Attempt. |
| Pause | Client Record, SMS | No new SMS Invoice while paused. REPRENDRE resumes. |
| STOP | SMS | Ack: facultatif off; factures continuent. Second STOP ≤24h: ANNULER instructions only. |
| ANNULER / Cancel | SMS, Client Record | Plan stopped; open Obligations still due. |
| Write-off | Encaissements | Merchant + Staff PIN. Transactional notice to Customer. Not Customer self-serve. |
| Session expired | Checkout | "Ce lien a expiré. Demandez un nouveau message." No collect. |
| OTP expired / wrong | Checkout, SMS | "Code expiré ou incorrect. Réessayez." No password fallback. |
| Duplicate MSISDN | Client Record create | Explicit rejection: this number already has a Client Record. |
| KYC hold | Accueil, Paramètres | Banner; payout actions disabled; invite/invoice remain. |
| Offline / 3G fail | Portal, Checkout | Toast: "Réseau faible. Réessayez." No silent debit. SMS path remains the fallback. |
| Sandbox | Checkout, Développeurs | Visible Sandbox chip. Fake Rail outcomes. No live Orange. Sandbox collected does not count toward take-rate or KYC sums. |
| Shared device | Portal | Staff PIN offer at end of sitting. No auto-logout animation; PIN gates money actions. |

## Interaction Primitives

- **Tap to act.** Primary target ≥ 44px. No hover-only affordances on `sm`.
- **MSISDN-first input.** Numeric keypad, CountryPack parse, show normalized `+226 ···` on blur.
- **OTP** — 4–6 digits, auto-submit when complete. **[ASSUMPTION]** 6 digits.
- **Staff PIN** — numeric pad, same target size. Gates listed actions only.
- **SMS verbs** — exact tokens, accent-insensitive. Unknown inbound → short AIDE. After STOP, AIDE-equivalent only if Customer originated the text.
- **Back** on Checkout steps: allowed; never collects.
- **One sheet deep.** Staff PIN replaces the confirm sheet.
- **Banned:** infinite scroll on Receivables Book (paginate), swipe-to-shame, public late leaderboard, drag-and-drop, English toggle on Checkout, card PAN fields, Customer password create, platform `*shortcode#` tutorial.

## Accessibility Floor

Behavioral. Visual contrast lives in `DESIGN.md`.

- WCAG 2.1 AA on Portal and Checkout where practical.
- Tap / click targets ≥ 44px.
- French `aria-label` on every icon-only control (Payout Clock info, overflow, PIN keys).
- Status never by color alone — word + icon + ink.
- Focus ring visible against `{colors.surface}` (shadcn `ring` + `{colors.primary}`).
- Screen reader announces surface on navigate: "Accueil, carnet des encaissements" / "Paiement, étape consentement, 2 sur 4."
- Reduce Motion: skip skeleton shimmer; instant state text.
- SMS verbs remain first-class for low-literacy and feature phones (NFR-8). Hosted web is equal, not superior.
- Do not trap focus in marketing chrome. Checkout is the page.

## Responsive & Platform

| Breakpoint | Portal | Checkout |
|---|---|---|
| `< md` (default) | Bottom tabs: Accueil · Clients · Plans · Encaissements · Paramètres. Développeurs inside Paramètres. First sitting full-width, no tabs until Accueil. | Phone column, 16px gutters. |
| `md`–`lg` | Tabs may move to top. Still one column, `{spacing.max-portal}`. | Same phone column centered. |
| `≥ lg` | Top nav six items including Développeurs + 720px column. Cybercafé PC sitting must complete without hover. | Same phone column — never a wide marketing split. |

SMS has no breakpoint. Feature-phone wrapping must keep the six invoice answers intact in the first four lines when possible.

No PWA / offline-write in MVP. **[ASSUMPTION]** A failed save does not keep a ghost Obligation.

## Inspiration & Anti-patterns

- **Lifted from Stripe Checkout:** one hosted page, Session URL, Merchant name as the trust header, developer Sandbox that matches live objects. Reject Stripe's card-first English `$` chrome.
- **Lifted from neighbourhood shop ledgers:** named person, amount, day, paid/not paid. The Accueil is a book, not a funnel dashboard.
- **Lifted from operator SMS:** short verbs, receipt as proof you can show a spouse.
- **Rejected — neon African-fintech gradients and flag chrome.** Trust is local name, not patriotism-as-UI.
- **Rejected — auto-debit / Flutterwave-style "subscriptions" copy.** BF MoMo is payer-approved each time.
- **Rejected — public defaulter wall, streak, shame ranking.**
- **Rejected — Customer password account / consumer Super-App home.**
- **Rejected — native Merchant apps** for informal owners. Mobile web first.
- **Rejected — cash as a second-class apology.**

## Shame-free collections

Private, factual, Merchant-to-Customer. No list titled *Mauvais payeurs*. Failure SMS does not claim the Subscription is cancelled. Late fee appears as a line on the next SMS Invoice, not as a public badge. Write-off is quiet and transactional. Mariam's gym Pause is a first-class verb, not a penalty.

## Payout Clock

Always two clocks, never one "paid" blob:

1. **Client payé** — Rail `collected` or Cash Mark-paid. Immediate.
2. **Vous recevez** — settlement target T+1, labeled *cible, pas une garantie*. If late: *en attente de règlement* + last Rail status.

Cash Mark-paid: Client payé and Vous recevez = déjà reçu. Sandbox: clocks may simulate; copy says Sandbox. Reversal: Client payé must not remain.

## CountryPack / locale

BF pack drives: French strings, `Africa/Ouagadougou` due dates, XOF scale 0, MSISDN parse, Orange prefix rules, KYC threshold 500 000 F CFA / calendar month, holidays via CalendarPort. No `if-Burkina` copy forks in components — switch the pack, not the JSX. Second-country production is out of UI scope.

## Key Flows

### Flow 1 — UJ-1 Awa enrolls her first weekly client (cybercafé PC + shared Android)

Protagonist: **Awa**, weekly cleaner, Ouaga; notebook roster; no tax ID.

1. Awa opens the Merchant Portal on a cybercafé PC. Signup: name **Awa**, MSISDN, neighbourhood **Ouaga — Dapoya**. OTP on her phone.
2. Plan: **2 000 F CFA**, weekly, due **vendredi**, late fee none, collection preference Orange + espèces.
3. Payout Destination: her Orange MSISDN.
4. Client Record: **Fatou** + Fatou's MSISDN. Confirm send.
5. Invite SMS leaves. Accueil shows Fatou as **En attente de consentement** and the next Friday — not as money owed.
6. Portal offers **Staff PIN** before she leaves the machine (nephew uses the same phone at home).
7. **Climax:** Accueil is a book with one named line, pending consent, empty owed, Payout Clock quiet. First sitting done in one breath. Later the same day she can add another Client Record in under a minute.

Failure: OTP timeout → resend, stay on Signup. Duplicate MSISDN → explicit reject. She walks away without Staff PIN → money actions later ask her to set one.

Cash later (same protagonist): Fatou pays 2 000 F CFA in hand Friday. Awa opens the Fatou row → **Marquer payé (espèces)** → Staff PIN → SMS Receipt. Payout Clock: Client payé · Vous recevez = déjà reçu.

### Flow 2 — UJ-2 Fatou pays this cycle from one French SMS

Protagonist: **Fatou**, Awa's client; feature phone or 3G Android; no email; shows SMS to her husband.

1. Fatou receives the Invite, then — after she consents — an SMS Invoice named **Awa — Nettoyage** with the six answers.
2. She replies **PAY** (or opens the Magic Link if she has a browser).
3. Identify: OTP over SMS. No password.
4. Consent (first time) states reminder-then-pay, next due date, STOP ≠ ANNULER, shared-phone risk.
5. Method: Orange Money. Confirm hand-off: she enters the Orange Rail-wallet confirm (operator USSD / in-app — not a platform shortcode).
6. **Climax:** One SMS Receipt. Shop name and amount match what she can show her husband. Silence after.

Failure: Orange does not confirm → Obligation remains; failure SMS; Awa sees Failed-collection Retry Inbox. Fatou can PAY again or pay Awa in cash.

Post-pay verbs: SOLDE, PAUSE, REPRENDRE, AIDE, ANNULER. STOP ≠ Cancel.

### Flow 3 — UJ-3 Adama ships hosted checkout from Sandbox

Protagonist: **Adama**, BF SaaS developer, Ouaga.

1. Same Merchant account type as Awa. **Développeurs** is visible immediately. Sandbox always on.
2. He creates Sandbox keys, a Hosted Checkout Session (MSISDN + Plan), redirects to the Hosted Checkout Page.
3. Page is French, F CFA, no cards, no English, no `$`. Trust chrome is *his* product name + neighbourhood.
4. He walks identify → consent → method → confirm against **SandboxRail**.
5. **Climax:** Webhook `collected` then `settled` with the same Obligation / Payout Clock objects he will see live. Orange PIN never touched his servers. Take-rate still 0 — sandbox collected does not count.

Failure: Webhook replay → idempotent token, no second debit. Expired Session → no collect.

### Flow 4 — UJ-4 Awa recovers a failed Friday collect

Protagonist: **Awa**. Orange timeout or Fatou cancelled Rail-wallet confirm.

1. Accueil / Encaissements shows Fatou, **Échec**, last Rail status, timestamp. Obligation still open.
2. Staff PIN (shared device). She taps **Réessayer** (new Collection Attempt + SMS) *or* **Marquer payé**.
3. Fatou already received the failure SMS (does not say cancelled).
4. **Climax:** Receivables Book is honest — owed until collected, skipped, or Written-off. Payout Clock updates only when money or cash is real.

Write-off (rare): Merchant + Staff PIN, transactional notice, not Customer self-serve.

### Flow 5 — UJ-5 Awa hits the soft KYC gate

Protagonist: **Awa**. Cumulative successful collections > 500 000 F CFA in the calendar month, or payout destination change, no tax ID.

1. Accueil and Paramètres show the KYC hold banner: **Paiements retenus — les factures continuent.**
2. She uploads NINEA/RCCM or national ID from Paramètres.
3. **Climax:** She understands why money is held — not a silent freeze. Invites and SMS Invoices still send.

**[ASSUMPTION]** Exact upload field checklist remains ops-owned; UX shows three document types and a pending-review state.

## Open items

| Item | Tag | Notes |
|---|---|---|
| Public brand wordmark | [ASSUMPTION] | Merchant name is the hero until locked |
| Exact SMS character counts / sender ID | open | Drafts above are field-complete |
| Late-fee compounding after many overdue cycles | open (PRD §8 UX) | First late fee = one line on next SMS Invoice |
| Merchant email backup auth | open (PRD §8 UX) | Not in MVP surfaces |
| KYC document checklist | [ASSUMPTION] | NINEA / RCCM / CNIB |
| OTP length | [ASSUMPTION] | 6 digits |
| Moov/Wave visibility | [ASSUMPTION] | Hidden until live |
| Dark mode | [ASSUMPTION] | Out of MVP |
