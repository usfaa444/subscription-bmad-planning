---
title: Product Brief — Sarati
status: complete
created: 2026-09-17
updated: 2026-09-18
---

# Product Brief: Sarati

Founder planning brief for a greenfield receivables-and-consent platform. Purpose: lock the product stance so a PRD can be written next. Not a VC deck.

## Product identity

| | |
| --- | --- |
| **Product name** | **Sarati** |
| **Etymology** | Dioula/Jula — *sarati* means agreement / contract / terms (trust + subscription/billing fit) |
| **Primary domain** | **sarati.net** (purchased by the product owner) |
| **Market** | Burkina Faso first; multi-country / multi-currency architecture later |
| **Planning repo** | https://github.com/usfaa444/subscription-bmad-planning |
| **Local folder slug** | `bf-recurring-payments` (unchanged; document content uses Sarati) |

## Executive Summary

**Sarati** is a Burkina Faso–first platform that lets any business collect weekly or monthly money without building a billing stack. A cleaner, tutor, gym, school, mosque, or small app should enroll a client by phone, send a bill, and get paid on the agreed day.

It is **one product with two skins**, not two systems. App merchants create a Hosted Checkout Session (Stripe-like URL + webhook). Offline merchants use a web portal and SMS. Both share the same consent and checkout state machine: identify → consent → method → confirm.

**Recurring here means customer-authorized pay-on-reminder**, never a merchant-initiated wallet pull. The live path is invoice → SMS → customer confirms → Orange Money debit → merchant credit. Cash mark-paid is first-class so the ledger stays complete when someone pays in hand. Consent copy written now must also cover a later account-to-account debit.

Launch is Burkina Faso, French-first, XOF, with MSISDN as the customer key. Architecture sockets for other rails, countries, and currencies exist from day one. No second-country production build in this version.

## The Problem

There is no simple way for a Burkinabè business to set up subscriptions or recurring collection. An offline owner chases clients on WhatsApp and in person. An app team would have to assemble identity, invoices, dunning, mobile-money quirks, receipts, and a ledger from scratch.

Existing MoMo checkout aggregators (CinetPay, Flutterwave, PayDunya, Hub2, and similar) sell **one-shot pay-ins**. They do not give the merchant a living who-owes-what book, a standing customer authorization, SMS-first enrollment, or cash as an equal collection method. Orange Money remains the largest wallet; Moov is second; Wave is live and growing. None of that is a receivables operating system.

The status quo costs missed cycles, shameful chasing, no proof when a client says “I already paid,” and no way to collect on time when the customer is not in the shop.

## The Solution

Build the **receivables-and-consent OS** and render it twice.

**Offline / portal.** The merchant registers in one sitting (under 15 minutes): business name and neighbourhood, a plan (frequency, due date, amount, late-fee policy), and clients by name + phone (under 60 seconds). The client gets an SMS that states what, to whom, how often, how much, and by when — plus how to pay. No customer password: magic link or OTP only.

**Online / API.** The merchant creates a checkout session; the customer lands on a hosted French MoMo page; on clear, the merchant app receives a webhook. Wallet secrets and PIN flows never sit on the merchant’s servers.

Under both skins the ledger separates **what is owed** from **how it is collected**. The customer picks a wallet at pay time, not a locked rail at subscribe. [ASSUMPTION] MVP may reach Orange via a local merchant API or a BF aggregator; the adapter hides that choice. Remaining MVP mechanics are in Scope.

Never market auto-debit. Show the next due date before it happens. Silence means success; ping on failure or change.

## What Makes This Different

The near-term advantage is **execution in this market**, not a proprietary rail.

- **Receivables OS, not checkout.** Competitors move money once. This product keeps the obligation, the mandate, the reminder, and the proof.
- **One consent protocol.** Hosted web, SMS verbs, and later USSD/IVR are renderers of the same machine. Feature phones are first-class in the model, even if the first live paths are SMS + hosted web.
- **Cash is the on-ramp.** Informal merchants enter by marking cash paid. Graduating those payers to MoMo is the growth loop.
- **Deferred moat.** Consented mandates become the switch when PI-SPI debit is productizable. The scheme is moving (launch 2025; Business APIs homologated Sep 2026; sandbox exists) — that makes sockets urgent, not live debit in-scope. Vacant `DirectDebitPort`; consent that stays valid later. Detail in `addendum.md`.
- **Trust artifacts over feature sprawl.** Named SMS, matching shop name, a receipt that looks official, a visible “customer paid / merchant received” clock. Settlement finality is the product.

## Who This Serves

**Primary — offline portal merchants.** Cleaners, tutors, private schools, gyms, mosques, compound / rent-like collectors. One person, often on a shared Android or a cybercafé PC. Success: stop hiring a collector; see who is late; money arrives or cash is marked; first client enrolled in under a minute.

**Secondary — developers and app merchants.** Ouaga (and later Abidjan) builders who want a Stripe-shaped session, webhook, and sandbox. Success: ship checkout in a week, not a quarter. They are the online distribution channel, not a second product.

**Customers.** People who want to stay in good standing with a trusted provider without walking to a kiosk every cycle. Shared phones are normal; the payer may not be the beneficiary (family pay is next, not MVP). French-first; no English, `$`, or card logos on first-run screens.

## Success Criteria

| Signal | First-version bar |
| --- | --- |
| Merchant sitting | Portal merchant live (plan + first client + first SMS) in **< 15 minutes** |
| Client register | Name + MSISDN only, **< 60 seconds** |
| Collection | First **successful** Orange or cash-marked cycle; obligation remains if a rail fails |
| Identity / locale | MSISDN key; French-only first run; amounts in **XOF / F CFA**; zero card chrome |
| Trust | SMS invoice + SMS receipt; payout clock visible; no double-charge on retry |
| Economics | Merchant **pays nothing until first successful collection**; then a take-rate |
| DX | Create session → hosted URL → webhook on clear; sandbox usable without live Orange |

[ASSUMPTION] Take-rate is a modest percentage, amount TBD. Treat the number as an open PRD input, not a locked price.

## Scope

**In (MVP floor).** Grouped from the brainstorm MUST list. NEXT / LATER matrices are in `addendum.md` §3.

- **Checkout + portal (one machine):** Hosted Checkout Session API + webhooks; French MoMo hosted page (no cards); merchant portal for plans, clients by phone, SMS invite; magic-link / OTP identity; staff PIN on shared devices.
- **Collect:** Reminder-then-pay as the only live path; Orange Money as the sole live rail behind an adapter also designed for Moov and Wave; cash mark-paid; idempotent collection tokens; failed-collection retry inbox; basic pause / skip.
- **Ledger + proof:** Plan separated from collection method; SMS invoice (what / whom / often / how much / by when); SMS receipt.
- **Locale + sockets:** CountryPack skeleton (BF + XOF + MSISDN parsing — no `if-Burkina`); sandbox; future-debit-capable consent copy; zero English / `$` / card-logo defaults.
- **Motion:** Sub-15-minute onboarding; take-rate after first success. A first bill is the same primitive as a subscription cycle (obligation + reminder + pay). That is the send-a-bill wedge, not a second product.

**Out of this version:** Live PI-SPI debit; auto-debit marketing; a second-country production pack; live Moov / Wave; native merchant apps; cards; holding customer balances if pass-through is viable; a payment-institution license. Full later list in `addendum.md` §3.

## Vision

If this works, the platform becomes the default way a West African small business keeps a receivables book and a customer keeps a standing authorization — first in Burkina Faso, then across UEMOA via CountryPacks, then further. When PI-SPI (or any A2A scheme) can carry a consented debit, merchants should not rewrite their integration. The story we tell is **on-time collection**, not autopilot emptying of wallets.
