---
title: Versions / reality-check review of architecture spine
date: 2026-09-17
lens: committed decisions web-researched vs training-data assertion
spine: _bmad-output/planning-artifacts/architecture/architecture-bf-recurring-payments-2026-09-17/ARCHITECTURE-SPINE.md
checked_against:
  - https://nodejs.org/dist/latest-krypton/
  - https://nodejs.org/uk/blog/release/v24.21.0
  - https://registry.npmjs.org/{next,typescript,react,@nestjs/core,@nestjs/bullmq,bullmq,pnpm,turbo,@opentelemetry/sdk-node,@opentelemetry/api}/latest
  - https://www.postgresql.org/docs/18/release-18-6.html
  - https://github.com/redis/redis/releases/tag/8.10.1
  - https://fly.io/docs/flyctl/mpg-create/
  - https://fly.io/docs/mpg/
  - https://fly.io/docs/reference/regions/
  - https://fly.io/mpg
  - https://fly.io/docs/upstash/redis/
  - https://docs.nestjs.com/observability/overview
  - https://github.com/nestjs/nest/releases/tag/v12.0.0
  - https://github.com/nestjs/nest-cli/issues/3479
  - https://nextjs.org/docs/app/api-reference/config/next-config-js/useTypeScriptCli
  - https://github.com/docker/compose/releases
  - https://docs.wave.com/business
  - https://www.orange.bf/business/fr/orange-money-paiement-en-ligne.html
verdict: FAIL — most npm pins exist and match `latest`, but the spine's "Verified 2026-09-17" claim overstates what was checked. Production host versions, Nest/Next/turbo starter defaults, and TypeScript 7 fit were not reality-checked.
---

# Review — Versions, existence, fit, starter defaults

Checked 2026-09-17 against npm registry, official docs, and Fly.io. The spine Stack table (lines 200–220) claims:

> Verified 2026-09-17 against npm registry, postgresql.org, Redis GitHub, Node.js release notes, Fly.io regions.

Memlog records those lookups. That is **registry-latest**, not **host-and-starter fit**. The holes below are the ones a builder would hit on day one.

## Verdict

**FAIL.** Named products still exist. Most exact pins are real and current. Three committed pairings were not web-checked for *fit* and will mislead implementation:

1. PostgreSQL **18.6** + production **Fly.io Managed Postgres** (MPG does not offer 18).
2. TypeScript **7.0.2** as the single TS version (Nest CLI cannot build on it; `nest new` defaults to TS 6).
3. Redis **8.10.1** as if it were the Fly production Redis (Fly Redis is **Upstash**, version unpublished; Fly docs warn BullMQ off pay-as-you-go).

Greenfield starter defaults for Nest 12, `create-next-app`, and `create-turbo` are also unrecorded.

---

## Stack table — live check (2026-09-17)

| Spine pin | Live source | Status |
| --- | --- | --- |
| Node.js 24.21.0 LTS (Krypton) | `nodejs.org/dist/latest-krypton/` lists `node-v24.21.0-*` dated 08 Sept 2026; [v24.21.0 notes](https://nodejs.org/uk/blog/release/v24.21.0). Node 26 is Current, LTS Oct 2026. | **Match.** Active LTS. Nest 12 needs ≥20.19 or ≥22.12; Next 16 needs ≥20.9. 24 LTS fits. |
| TypeScript 7.0.2 | `registry.npmjs.org/typescript/latest` → `7.0.2`. | **Exists / latest.** **Fit fail** — see Finding 2. |
| `@nestjs/core` 12.0.3 | npm latest 12.0.3, published 2026-09-15. | **Match.** |
| `@nestjs/bullmq` 12.0.0 | npm latest 12.0.0, published 2026-08-27. Peers: Nest `^10 \|\| ^11 \|\| ^12`, bullmq `^3–^6`. | **Match / fits Nest 12 + BullMQ 6.** |
| Next.js 16.3.5 | `registry.npmjs.org/next/latest` → `16.3.5` (tarball `next-16.3.5.tgz`). Website/search caches still show 16.3.4. | **Match** (registry, not the website). |
| React 19.3.0 | npm latest 19.3.0 (2026-09-09). Next 16 peer `^18.2 \|\| ^19`. | **Match / fits.** |
| PostgreSQL 18.6 | postgresql.org release 18.6 on 2026-08-13; 19 is Beta 3. | **Exists as OSS latest.** **Host fail** — see Finding 1. |
| Redis 8.10.1 | github.com/redis/redis `8.10.1` (2026-08-17); Docker `redis:latest` = 8.10.1. | **Exists as OSS/Docker latest.** **Host fail** — see Finding 3. |
| BullMQ 6.3.4 | GitHub tag 6.3.4 (2026-09-01). npm `latest` is **6.3.6** (2026-09-14). | **Exists, stale by two patches** on the claimed verify date. |
| pnpm 12.4.2 | npm latest `12.4.2`. Website/blog still talk 12.2–12.3 / npm page cache on 11.x. | **Match** (registry). |
| `turbo` 2.10.13 | npm latest `2.10.13`. Website/search caches still show 2.10.12. | **Match** (registry). |
| `@opentelemetry/sdk-node` 0.222.0 | npm latest 0.222.0 (2026-08-31). Package is **experimental 0.x**; 0.222.0 has breaking fail-fast changes. | **Match / experimental.** Nest 12 first-party path is `@nestjs/observe` — see Finding 4. |
| `@opentelemetry/api` 1.9.x | npm latest **1.9.1**. sdk-node peer `>=1.3.0 <1.10.0`. | **Match.** |
| Fly.io ams + Managed Postgres | [Regions](https://fly.io/docs/reference/regions/): `ams` is a **gateway**. [fly.io/mpg](https://fly.io/mpg) lists Amsterdam. `jnb` exists, not a gateway, not on the MPG marketing list. | **ams + MPG exist.** PG major on MPG is not 18. |
| Docker Compose v2 | github.com/docker/compose latest release **v5.3.1** (2026-07-07). v2 is the historical plugin generation (`docker compose` vs `docker-compose`). | **Product exists. Pin is outdated / vague.** |

---

## Findings

### 1. PostgreSQL 18.6 cannot be the production MPG version — AD-14 + Stack

**Location:** Stack table; AD-14; fly diagram (`Managed Postgres`).

**Trigger:** Spine binds production to Fly.io MPG **and** pins PostgreSQL **18.6**. Official `fly mpg create` (fetched 2026-09-17):

> `--pg-major-version` … **Supported versions are 16 and 17. (default 16).**

[Managed Postgres overview](https://fly.io/docs/mpg/) still describes “trusted extensions included in the **default Postgres 16** distribution” and lists “Security patches and version upgrades” under *What’s not there yet*. Community asked for PG 18; Fly has shipped 17, not 18.

**Guard:** Split the pin: local Compose `postgres:18.6` (or 17 to match prod) vs production MPG **16 (default) or 17**. Do not tell builders “Postgres 18.6” is the Fly live default. Prefer MPG 17 if they want newest *hostable* major.

**Consequence:** A builder following the stack table will compose PG 18 locally and cannot provision the same major on Fly. MPG has no in-place major upgrade path in current docs.

### 2. TypeScript 7.0.2 is npm latest and breaks Nest greenfield — Stack + implied starter

**Location:** Stack `TypeScript 7.0.2`.

**Trigger:** Version is real. Fit was not checked against the starters this spine leans on.

- NestJS 12 `nest new` / `@nestjs/schematics` 12 defaults to **TypeScript 6**, NodeNext, ESM, Vitest, oxlint, Rspack. Peer: `typescript >= 6.0.0`. Upgrade schematic bumps TS to `^6.0.0`.
- `@nestjs/cli` **cannot** `nest build` / `nest start` / `--watch` on `typescript@7.0.2` — TS 7 dropped the programmatic Compiler API (`getParsedCommandLineOfConfigFile`, `createProgram`). [nestjs/nest-cli#3479](https://github.com/nestjs/nest-cli/issues/3479) tracks real support; current guard is “install TypeScript 6”.
- Next.js 16.3 can type-check with TS 7 only via **`experimental.useTypeScriptCli: true`**. Without it, `next build` treats TS 7 as missing (no `typescript/lib/typescript.js`). Documented minimum remains TS **5.1.0**; default checker is still the JS API (TS 6.x).

**Guard:** Pin **TypeScript 6.x** as the Nest/`nest build` compiler (or the Microsoft dual-install: `typescript` = `@typescript/typescript6`, `tsc` 7 only for `tsc --noEmit`). If TS 7 is kept for Next, record `experimental.useTypeScriptCli`. Do not list a single `7.0.2` as if it were the greenfield default.

**Consequence:** `nest new` + spine pin = a repo that cannot `nest build` until someone discovers the CLI hole.

### 3. Fly production Redis is Upstash, not Redis 8.10.1 — AD-14 diagram

**Location:** AD-14 mermaid `FRD[(Redis)]` under `fly [Fly.io ams]`; Stack `Redis 8.10.1`.

**Trigger:** 8.10.1 is current **Redis OSS** and `docker.io/library/redis:latest`. Fly’s live Redis product is **[Upstash for Redis](https://fly.io/docs/upstash/redis/)** via `fly redis create`. Docs do not publish an OSS Redis version. They explicitly warn:

> If you're using Sidekiq, **BullMQ** or similar software, consider switching … to a **fixed price plan** to avoid running up your pay-as-you-go bill.

Replicas forward writes to primary; PAYG is $0.20 / 100k commands. BullMQ workers poll.

**Guard:** Name **Upstash (Fly Redis)** for production, require a **fixed** plan for BullMQ, pin Redis **8.10.1 only for local Compose**. Confirm ioredis / BullMQ 6 compatibility with Upstash (not assumed from OSS 8.10.1).

**Consequence:** Builders will expect Redis 8.10.1 semantics and self-hosted HA on Fly. They will get a different product, version, eviction model, and a PAYG bill if they follow the diagram blindly.

### 4. Nest 12 / Next 16 / create-turbo live defaults were not recorded

**Location:** Structural Seed; Stack; AD-1 (`apps/api` Nest + workers); AD-14 OTel.

**Trigger:** Greenfield spine, no existing repo conventions. Architecture skill requires checking starter defaults. Spine names Nest, Next, turbo, pnpm — not what those CLIs emit today.

| Starter | Live default (2026-09) | Spine |
| --- | --- | --- |
| `nest new` (v12) | **ESM** (`"type": "module"`), **Vitest**, **oxlint**, **Rspack**, **TypeScript 6**, optional `--observe` → `@nestjs/observe` | Silent. Pins raw `@opentelemetry/sdk-node` 0.222.0. |
| `create-next-app@latest` | TypeScript, Tailwind, ESLint, **App Router**, Turbopack, `AGENTS.md`, React 19 + Next 16 | Portal + checkout implied; App Router / Tailwind / TS 7 flag unstated. |
| `create-turbo@latest` | Two **Next.js** apps (`web`, `docs`) + `@repo/ui` + eslint/tsconfig packages | Seed is `apps/api` + `portal` + `checkout` + `packages/domain`. Official turbo starter does **not** include Nest. |

Nest 12 release headline observability is **`@nestjs/observe`** (hooks `NestFactory.create({ instrument })`, covers HTTP + **queue consumers**). Raw `sdk-node` 0.222.0 is valid OTel but is **not** the Nest 12 starter default, is still 0.x experimental, and 0.222.0 ships breaking fail-fast on declarative/env resource detectors.

**Guard:** Record: hand-roll the monorepo (do not `create-turbo` basic); `nest new --type esm` (or cjs, chosen); Next App Router; either `@nestjs/observe` **or** raw OTel with a reason. Pin OTel 0.222.0 only if Observe is rejected.

**Consequence:** Two builders will fork on ESM vs CJS, Vitest vs Jest, Observe vs sdk-node, and whether to start from `create-turbo`.

### 5. Docker Compose “v2” and BullMQ 6.3.4 are stale relative to the verify date

**Location:** Stack last two rows / BullMQ row.

**Trigger:** Claimed verification 2026-09-17.

- Compose current release is **v5.3.1** (2026-07-07). “v2” means the Go plugin (`docker compose`), not a version a lockfile can pin.
- BullMQ npm latest on 2026-09-14 is **6.3.6**. 6.3.4 is 16 days old. `@nestjs/bullmq@12` peers allow `^6`, so 6.3.6 is in range.

**Guard:** Write `Docker Compose` (plugin / Compose spec) without a dead major, or pin `>=v2` / `v5.x`. Bump BullMQ to **6.3.6** or say `^6.3`.

**Consequence:** Minor for Compose (command still works). BullMQ pin signals the table was snapshotted before 2026-09-14, undermining the 2026-09-17 verification sentence.

---

## Named technologies — existence and fit

| Name | Exists? | Fits this spine? | Notes |
| --- | --- | --- | --- |
| NestJS 12 | Yes | Yes | Modular monolith + DI + `@nestjs/bullmq`. Hexagonal is a layering choice, not a Nest default. |
| Next.js 16 | Yes | Yes | App Router BFF/portal/checkout. |
| React 19.3 | Yes | Yes | Next 16 peer. |
| PostgreSQL | Yes | Yes as engine | **18.6 does not fit Fly MPG.** |
| Redis / BullMQ | Yes | Yes locally | Fly = Upstash; use fixed plan. |
| pnpm 12 + turbo 2.10 | Yes | Yes | Official turbo example is Next-only. |
| OpenTelemetry JS | Yes | Yes | sdk-node still experimental; Nest 12 prefers `@nestjs/observe` for Nest-shaped traces. |
| Fly.io `ams` + MPG | Yes | Yes as region/host | MPG majors 16/17, default 16. `jnb` correctly Deferred (no MPG on marketing list; not a gateway). |
| ULID (Crockford) | Yes | Yes | `ulid@3.0.2` (2025-11-30), Node 18+. Unpinned in Stack — convention only. |
| HMAC-SHA256 webhooks | Yes | Yes | Wave itself uses HMAC-SHA256 (`Wave-Signature`). Stripe-like `t=`/`v1=` is a convention, not a library. |
| Orange Money | Yes | Yes as live BF rail | [Orange BF Paiement en ligne](https://www.orange.bf/business/fr/orange-money-paiement-en-ligne.html) is a **local contract + NDA** product. Pan-African [Orange Developer WebPay](https://developer.orange.com/apis/om-webpay) country list in 2026 still omits BF. Spine correctly leaves counterparty as adapter config. |
| Wave | Yes | Stub-only is correct | [docs.wave.com/business](https://docs.wave.com/business) live Checkout/Payout APIs; Wave operates in BF. Not live in MVP — fine. |
| Moov Money | Yes as wallet | Stub-only is correct | No comparable public self-serve merchant API in 2026 sources; aggregator/gateway typical. Spine’s “designed stub” matches reality. |
| Africa/Ouagadougou | Yes | Yes | IANA TZ for BF. Not independently re-fetched this pass; standard and stable. |
| Docker Compose | Yes | Yes | Current line is v5, not v2. |

SMS vendors (Africa’s Talking / Twilio / BF aggregator) are Deferred — correct; no false pin.

---

## What the spine *did* confirm (do not re-litigate)

These look like real 2026-09 lookups, not training-data fossils:

- Node 24 LTS Krypton vs Node 26 Current.
- Nest 12.0.3 (days old), Next 16.3.5, React 19.3.0, pnpm 12.4.2, turbo 2.10.13.
- Fly `ams` as gateway + MPG region; `jnb` deferred for MPG/gateway reasons.
- `@nestjs/bullmq` 12 aligned with Nest 12 and BullMQ 6.

The failure is **stopping at “package exists”** and not checking **host major, CLI build, and starter emit**.

---

## Unconfirmed / still asserted

| Claim | Why it is not web-closed |
| --- | --- |
| Single TypeScript 7.0.2 for the whole monorepo | Conflicts with Nest CLI + Next default checker. |
| Postgres 18.6 in production | Conflicts with `fly mpg create`. |
| Redis 8.10.1 on Fly | Fly Redis version unpublished; product is Upstash. |
| Docker Compose v2 | Current release v5.3.1. |
| Raw `@opentelemetry/sdk-node` as the ops default | Nest 12 ships `@nestjs/observe`; 0.222.0 is experimental. |
| ESM vs CJS for `apps/api` | Nest 12 default is ESM; spine silent. |
| `create-turbo` as implied monorepo seed | Official example is two Next apps. |
| ULID package / version | Convention only; `ulid@3.0.2` would be the current pin if they wanted one. |
| Orange BF transport (WebPay vs local API vs aggregator) | Deferred correctly; Orange.com WebPay country list still omits BF. |

---

## Required spine edits (if this lens is applied)

1. Split Postgres: **MPG 16 or 17 (default 16)** vs optional local 18.6 — or pin **17** everywhere.
2. Split TypeScript: **6.x for Nest CLI**; TS 7 only with Next `experimental.useTypeScriptCli` and a dual-install note.
3. Name **Upstash / `fly redis`** + **fixed plan** for BullMQ; keep 8.10.1 for Compose only.
4. Record Nest 12 emit: ESM (or chosen CJS), Vitest, oxlint, Rspack; choose `@nestjs/observe` or justify sdk-node 0.222.0.
5. Soften “Verified 2026-09-17 …” or extend it to Fly MPG majors + starter CLIs. Bump BullMQ to 6.3.6. Drop Compose “v2” as a version.
