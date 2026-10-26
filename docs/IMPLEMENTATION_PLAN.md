# Implementation Plan — Garda Jaya CCTV

**Document:** `docs/IMPLEMENTATION_PLAN.md`
**Version:** 1.0 — planning artifact
**Date:** 2026-10-26
**Status:** **Proposed — not approved. Authorises no implementation.**

**Sources of truth:** `PRD.md` v0.2.2 (requirements, constraints, phase intent) and `AGENTS.md` (working rules). This document adds *sequencing* and *file-level* detail only. It creates no requirement. Where this plan and the PRD differ, **the PRD wins** and this plan is the thing that is wrong.

**Stack assumed (per owner decision recorded in `PRD.md` §12.9 / §17 `OQ-013`):** Astro + `@astrojs/cloudflare` on Cloudflare Workers, TypeScript `strict`, Cloudflare D1, Cloudflare R2.

**Resource names supplied with this brief:** D1 database `gjdb`, R2 bucket `gj-cctv`, site hostname `cctv.artupski.com`. These are **names, not provisioning evidence** — see §2.

---

## 0. How to use this document

1. Read §2 (readiness) first. **If any gate row is not `Met`, no phase below may start** (`AGENTS.md` §14, `PRD.md` §18.1.1).
2. Read §5 (dependency map) before scheduling anything. Several phases are not linear.
3. Work phase by phase. Each phase has an **exit condition**; do not enter the next phase until it is satisfied.
4. Every work item must name the `FR-###` it implements and the `AC-###` that proves it (`AGENTS.md` §2). An item with no requirement is out of scope — stop and ask.
5. **This plan does not authorise:** scaffolding, dependency installation, resource creation, deployment, commits, or pushes. It is a proposal for owner approval.

### 0.1 Vocabulary

| Term | Meaning |
|---|---|
| `Cloudflare-independent` | Can be built, run and tested with **no Cloudflare account, credential or provisioned resource** — local `workerd` via `wrangler dev`, local D1/R2 emulation, unit tests, typecheck, lint, `wrangler deploy --dry-run`. |
| `Cloudflare-dependent` | Requires an owner-provisioned resource, credential, secret, DNS record, or a remote API call. Cannot be completed by an agent alone. |
| `Met` / `Open` / `Blocked` | Gate status. `Open` = no evidence either way. `Blocked` = known unmet and cannot be closed by an agent. |
| `Deferred` | An acceptance criterion that this phase cannot honestly verify; it is re-assigned to a later phase and listed in §7.1. |

---

## 1. Inputs and evidence used

| Input | Use |
|---|---|
| `PRD.md` v0.2.2 | Requirements (`FR`/`NFR`/`AC`), constraints, §18 phase intent, §18.1.1 gate, §13 schema outline, §13.2–§13.3 D1/R2 semantics |
| `AGENTS.md` | Hard constraints (§4), security (§5), SEO (§6), licence gate (§8), testing intent (§9), D1/R2 rules (§10), phase gate (§14) |
| `logo/` (4 PNGs) | Brand reference only — read-only, never edited (§10.5). Dimensions and gaps per PRD §10.1–§10.2 |
| Brief accompanying this task | Stack confirmation, resource names, phase frame (0–6) |

**Not used, and why:** the repository was not re-audited. `PRD.md` §2.6 and `AGENTS.md` §13 already record the tree state (four root entries, one commit). No new verification cycle was run.

---

## 2. Phase 0 — Readiness

### 2.1 Purpose and scope

Close, or explicitly declare unmet, the gates that must hold before the work they gate. Phase 0 produces **no code**. Its only output is a decision record plus the owner-supplied evidence that closes each row.

Phase 0 is **not** a repository audit and **not** a Cloudflare audit. Gate status below is transcribed from `PRD.md` §18.1.1 and §17, plus the resource names in the brief. Nothing was re-verified.

### 2.2 Evidence available in the workspace — and what is missing

| Evidence the brief refers to | Present in workspace? | Consequence |
|---|---|---|
| `.wrangler/verify-account.ps1` | **No** | Account-readiness evidence unavailable |
| `.wrangler/verify-zone.ps1` | **No** | Zone-readiness evidence unavailable |
| `.wrangler/verify-dns-r2.ps1` | **No** | R2 custom-domain / DNS evidence unavailable |
| `.wrangler/verify-plan.ps1` | **No** | Plan-tier evidence unavailable |
| `.wrangler/verify-raw.ps1` | **No** | Raw API evidence unavailable |
| Recorded results of those scripts | **No** | No gate row can be closed from workspace evidence |

The workspace contains exactly `.gitignore`, `AGENTS.md`, `PRD.md` and `logo/` (four PNGs). `.wrangler/` is absent and is gitignored by design (`AGENTS.md` §4.3). **No verification was re-run.** The rows below therefore stay `Open` rather than being guessed in either direction.

### 2.3 Gate table

| ID | Gate item (`PRD.md` §18.1.1) | Status | Evidence / note | Blocks |
|---|---|---|---|---|
| G0-1 | Framework signed off (`OQ-013`) | **Open** | Decision *recorded* as Confirmed (`PRD.md` §12.9, §13.4 `D-02`, §17). §18.1.1's own status note requires it "reflected in a document the owner has signed". The brief restates Astro + `@astrojs/cloudflare`, which is corroboration but not the signed artifact | Phase 1 scaffold |
| G0-2 | Cloudflare account ready, admin named | **Open** | No evidence; `verify-account.ps1` absent | Phase 1 remote config, Phase 4 remote apply, Phase 6 |
| G0-3 | Zone / domain active in that account | **Open** | Hostname `cctv.artupski.com` named in the brief; zone ownership unverified (`OQ-017` **Open**) | Phase 3 canonical URLs, Phase 6 DNS/TLS |
| G0-4 | D1 provisioned by owner, IDs recorded | **Open** | Database name `gjdb` supplied; `database_id` for production **and** preview not recorded | Phase 4 remote migrations, Phase 6 |
| G0-5 | R2 provisioned, custom domain attached or hostname agreed | **Open** | Bucket name `gj-cctv` supplied; media hostname not agreed (`D-24` **Open**) | Phase 3 image URLs, Phase 4 upload acceptance, Phase 6 |
| G0-6 | Image strategy decided (`OQ-014`) | **Open** | Answer recorded: **pre-generated variants** (`PRD.md` §17). Missing: the *variant specification* (width set, formats, max upload dimension, who generates them). Without it `FR-106`/`FR-115` are unimplementable | Phase 2 image component, Phase 4 upload pipeline |
| G0-7 | Password strategy decided after benchmark (`OQ-015`) | **Blocked** | PBKDF2-SHA256 recorded as chosen; **benchmark on the real runtime not performed** and §17 states it "remains mandatory". See the circular-dependency note in §8 `OD-03` | Phase 4 auth |
| G0-8 | Workers plan tier decided (`OQ-018`) | **Open** | Answer recorded: **start Free**; D1 primary location still open. Plan tier is decided, the location is not | Phase 5 performance, Phase 6 |
| G0-9 | PRD v0.2.2 approved as a document | **Blocked** | `PRD.md` still reads "Draft v0.2.2 — awaiting owner approval". `AGENTS.md` §2 forbids describing it as approved | **Everything** |

**Gate state: 0 of 9 met.** This matches `PRD.md` §18.1.1's own status note ("the gate therefore remains 0 of 9"). It is not a new finding — it is the recorded state.

### 2.4 Phase 0 work items

| # | Item | Owner | Class | Output |
|---|---|---|---|---|
| P0.1 | Owner signs the PRD status line off "Draft" (G0-9) | Owner | — | PRD status change (owner edit; **not** an agent edit) |
| P0.2 | Owner records the four answers already given (`OQ-013`, `OQ-014`, `OQ-015`, `OQ-018`) in a signed note or PRD amendment so G0-1/G0-6/G0-8 have artifact evidence | Owner | — | Signed note |
| P0.3 | Owner supplies account ID, zone ID, `gjdb` production + preview `database_id`, `gj-cctv` bucket name, and the media hostname | Owner | Cloudflare-dependent | Values for `wrangler.jsonc` and a secrets inventory |
| P0.4 | Re-attach the `.wrangler/verify-*.ps1` scripts and their recorded output, **or** state that they no longer exist | Owner | — | Gate evidence for G0-2..G0-5 |
| P0.5 | Fix the §13.4 rows whose status was flipped to Confirmed without recording the chosen value (see §9) | Owner | — | PRD correction |
| P0.6 | Approve or reject the sequencing deviation in §8 `OD-11` | Owner | — | Sequencing decision |
| P0.7 | Approve or reject the G0-7 benchmark-exception route in §8 `OD-03` | Owner | — | Unblocks Phase 4 auth |
| P0.8 | Agree the image variant specification (G0-6 detail) | Owner + implementer | — | Variant table |

### 2.5 Acceptance criteria for Phase 0

- Every row in §2.3 reads `Met`, with a named artifact or recorded value, **or** the owner has explicitly accepted the risk of starting the specific phase it gates.
- `PRD.md` §18.1.1 is updated by the **owner** to reflect the new state. An agent must not edit `PRD.md` to close a gate.
- §8 `OD-01`–`OD-05` are answered.

### 2.6 Testing and completion evidence

No tests. Evidence is the signed artifacts: PRD status line, the recorded decision note, the resource-ID table, and the restored verification output.

### 2.7 Risks and blockers

| Risk | Impact |
|---|---|
| Gate closed verbally but not in writing | Reproduces the condition `AGENTS.md` §2 warns about: a decision that exists only in chat history |
| Resource IDs transcribed wrong | `wrangler.jsonc` points at the wrong database; the classic path to writing preview data into production (`FR-195`) |
| Gate bypassed "just to scaffold" | Direct violation of `AGENTS.md` §14 and §4.10 |

### 2.8 Exit condition

All of G0-1…G0-9 `Met`, or the owner has signed a written, itemised exception naming exactly which phase may start without which gate. **No exception is assumed by this plan.**

---

## 3. Phase overview

| Plan phase | Name | PRD mapping | Class | Exit milestone |
|---|---|---|---|---|
| P0 | Readiness | §18.1 P0, §18.1.1 | — | **M0** Gate closed |
| P1 | Project foundation | §18.1 P1 | Mostly independent | **M1** Local skeleton runs |
| P2 | Public website | §18.1 P4 (see §8 `OD-11`) | Independent (2A) / data-bound (2B) | **M2** Public site reviewable |
| P3 | Content & SEO | §18.1 P5 (SEO half) | Mostly independent | **M3** SEO surface complete |
| P4 | Admin & data | §18.1 P2 + P3 | Local-independent, remote-dependent | **M4** Admin usable end-to-end locally |
| P5 | Testing & hardening | §18.1 P5 (perf/security half) + §18.3 | Independent / preview-dependent | **M5** Targets met on preview |
| P6 | Deployment readiness | §18.1 P6 | **Fully Cloudflare-dependent** | **M6** Launch-ready, **not deployed** |

**Phase-to-PRD drift is intentional and declared.** The PRD puts public pages at P4 and auth/DB at P2; this plan proposes public presentation earlier. See §8 `OD-11` — the deviation needs owner approval, and Phase 2's data-dependent acceptance criteria are re-assigned rather than claimed early.

---

## 4. Cloudflare-independent vs Cloudflare-dependent work

This split is the single most useful scheduling tool in the plan: it shows how far the project can progress while G0-2…G0-5 remain `Open`.

### 4.1 Can proceed with no Cloudflare resource, credential or account

- Extending `.gitignore`; tooling; Astro scaffold; `wrangler.jsonc` with **placeholder** `database_id`/bucket name for local emulation only.
- TypeScript config, lint config, CI workflow (`typecheck`, `lint`, `build`, `wrangler deploy --dry-run`).
- Design tokens, base layout, header/footer, 404 and error shells.
- All public route files and components, rendered from a **fixture adapter** behind the repository interface.
- Pure logic modules: slug normalisation, price-display branching, WhatsApp number normalisation, currency/date formatting, redirect-chain collapsing.
- SEO head component, `sitemap.xml` and `robots.txt` endpoints, JSON-LD builders.
- Admin shell markup, form validation modules, HTML sanitiser wrapper, CSRF token generation, PBKDF2 benchmark harness under local `workerd`.
- Migration `.sql` files **authored** (not applied remotely), applied to **local** D1 with `--local`.
- Unit and integration tests against local D1/R2 emulation, including `batch()` fault injection.

### 4.2 Requires Cloudflare resources, credentials or owner evidence

- Any real `database_id`, any `--remote` migration apply, any preview-environment binding.
- R2 bucket binding, bucket CORS, and the media custom domain (`FR-188`, `FR-190`).
- Cache purge on publish (`FR-125`, `FR-179`, `FR-186`) — needs a zone ID and an API token with cache-purge permission.
- Secrets: `wrangler secret put` for the session signing key and any API token (`FR-192`).
- DNS records and Worker custom domain for `cctv.artupski.com`.
- `observability` verification in a real environment (`FR-196`).
- Scheduled D1 export to R2 and the restore drill (`FR-197`, `AC-074`).
- Performance and cache-hit measurement on a preview deployment (`NFR-044`, `AC-076`).
- Preview/production isolation proof (`FR-195`, `AC-073`).
- Any deployment whatsoever.

### 4.3 The important nuance

Local D1 and local R2 are **emulated by Wrangler/Miniflare and need no account**. So Phase 4 is *developable and testable* end-to-end without Cloudflare — but not *acceptable* until the remote evidence exists. Do not let "it works locally" be reported as `AC-062`/`AC-064` passing; those criteria explicitly require the remote object and the custom domain.

---

## 5. Cross-phase dependency map

```
P0 ──► P1 ──┬──► P2A (public shell, tokens, static routes, 404)
            ├──► P3  (SEO head, sitemap, robots, JSON-LD, image component)
            └──► P4.1–P4.4 (schema, D1 access, batch helper, repositories)
                        │
                        ▼
                  P2B (catalog, detail, filters, homepage blocks, contact form)
                        │
                        ▼
                  P4.5–P4.14 (auth, admin UI, media, settings, publishing)
                        │
                        ▼
                       P5 ──► P6
```

| Dependency | Type | Note |
|---|---|---|
| P1 → P2, P3, P4 | Hard | No scaffold, no routes |
| P4.1–P4.3 → P2B | Hard | Public content surfaces read through the repository contract; the contract must exist before they can render real data |
| P2A → P2B | Soft | P2B reuses P2A's components; building P2B first would duplicate markup |
| P2A → P3 | Soft | SEO component wraps P2A's layout; P3 can start once `BaseLayout` is stable |
| P4.4 (password benchmark) → P4.5 (auth) | Hard | `AC-027` requires recorded parameters |
| P4.10 (settings) → P2B (WhatsApp CTA, contact block) | Hard | `AC-005`/`AC-012`/`AC-045` need CMS-configured values |
| P4.11 (publishing + `slug_history`) → P3 (301 handling) | Hard | Redirects cannot be verified before the history table is written to |
| P4.13 (preview tokens) → P3 (`noindex` rules) | Hard | `AC-053`/`AC-070` need a preview URL to exist |
| P3 → P5 | Soft | P5 verifies what P3 produced |
| P4 → P5 | Hard | Integration and e2e tests need the admin |
| P5 → P6 | Hard | No production config before the security/performance pass |
| P6 (DNS, custom domain) → P3 final verification | **Reverse** | Canonical URLs and R2 URLs cannot be finally verified before DNS exists — see §8 `OD-04` |

**Reverse dependencies are the scheduling trap.** Two acceptance criteria (`AC-049` canonical URLs, `AC-064` R2 custom domain) cannot be closed inside the phase that produces the code. They are declared `Deferred` and re-assigned to P6.

---

## 6. Milestones

| ID | Milestone | Definition | Gate |
|---|---|---|---|
| M0 | Readiness closed | §2.3 all `Met` or explicitly excepted | Phase 0 exit |
| M1 | Local skeleton runs | `wrangler dev` serves `/` and `/404`; typecheck, lint and build clean; `wrangler deploy --dry-run` succeeds; no remote resource touched | Phase 1 exit |
| M2 | Public site reviewable | Every route in PRD §5.1 renders from the fixture adapter at 320–1536 px; a11y checklist passes; owner has reviewed design and copy | Phase 2 exit |
| M3 | SEO surface complete | One canonical per page, unique metadata, sitemap, robots, JSON-LD validated, image component emits `srcset` | Phase 3 exit |
| M4 | Admin usable locally | Owner can log in, create/publish a product and an article, upload to emulated R2, change settings, rename a slug and see a 301 — all against local D1 | Phase 4 exit |
| M5 | Targets met on preview | `AC-060`, `AC-072`, `AC-076`, `AC-078` measured on a Workers preview deployment on the chosen tier | Phase 5 exit |
| M6 | Launch-ready | Backup exported and restore rehearsed, DNS plan documented, smoke-test script written, cost model written — **nothing deployed** | Phase 6 exit |

---

## 7. Phases

### Phase 1 — Project foundation

#### Purpose and scope

Produce a runnable, type-checked, Workers-native skeleton with the brand system, the localisation constants, and the error/404 shells. No content, no auth, no database queries. This phase establishes the conventions every later phase inherits — it is the only phase where "no conventions to follow" is still true.

#### Prerequisites and dependencies

- G0-1 `Met` (framework signed), G0-9 `Met` (PRD approved).
- Owner decision on `OD-12` (permission to extend the existing `.gitignore`).
- No Cloudflare resource required for the local path; a real `database_id` is required only for remote config.

#### Files and modules

| Path | Action | Purpose |
|---|---|---|
| `.gitignore` | **modify** | Must cover `.dev.vars`, `.env*`, `.wrangler/`, `node_modules/`, `dist/`, `.astro/`, `*.db`, `*.sqlite`. Currently one line. **Boundary note:** this planning task must not touch it; the extension is Phase 1 work requiring `OD-12` |
| `package.json`, lockfile | create | Pinned deps; scripts `dev`, `build`, `preview`, `typecheck`, `lint`, `test`, `db:migrate:local` |
| `astro.config.mjs` | create | `@astrojs/cloudflare` adapter; `output: 'server'`; `imageService: 'passthrough'` so no on-demand transformation is reachable (`FR-198`) — confirm the option name against the adapter docs at implementation time |
| `wrangler.jsonc` | create | `compatibility_date` pinned, `compatibility_flags: ["nodejs_compat"]`, `assets`, `observability.enabled`, `d1_databases` → `DB`, `r2_buckets` → `MEDIA`. Local path may use a placeholder `database_id`; remote path is blocked until G0-4 |
| `tsconfig.json` | create | `strict: true`, no `any` escape hatches |
| `worker-configuration.d.ts` | generate | `wrangler types` output; generated, committed, never hand-edited |
| `src/env.d.ts` | create | `App.Locals` typing for `runtime.env` |
| `src/lib/config/site.ts` | create | Single source for `id-ID` and `Asia/Jakarta` (`FR-016`, `NFR-012`) |
| `src/lib/config/bindings.ts` | create | Reads bindings, fails fast with a named missing binding (`FR-193`, `NFR-053`) |
| `src/lib/i18n/id.ts` | create | Centralised Bahasa Indonesia strings |
| `src/styles/tokens.css`, `src/styles/base.css` | create | Navy/blue/sky/white + derived neutrals; focus ring; reduced-motion; type scale |
| `src/layouts/BaseLayout.astro` | create | `lang="id"`, skip link, landmarks, `<slot />` |
| `src/components/public/{SkipLink,Header,Footer,WhatsAppAction,Container}.astro` | create | Shell pieces; WhatsApp action is a component so `AC-005` is testable once settings exist |
| `src/pages/index.astro` | create | Placeholder homepage that renders **nothing** rather than invented copy (`FR-013`, `FR-038`) |
| `src/pages/404.astro` | create | Bahasa Indonesia, nav home + WhatsApp CTA (`FR-005`, `AC-003`) |
| `src/pages/500.astro` | create | Generic error, no stack trace (`NFR-059`) |
| `src/pages/api/health.ts` | create | Returns binding presence booleans only — never values (`AC-061` (d)) |
| `public/favicon.ico`, `public/icon-192.png`, `public/apple-touch-icon.png` | create | **Derived** from `logo/Garda-Jaya-CCTV-Main-Logo-Only.png` on a square transparent canvas; never upscaled beyond native 300 px (PRD §10.3) |
| `eslint.config.js` | create | Flat config, TypeScript rules, no warnings tolerated in CI |
| `.github/workflows/ci.yml` | create | `typecheck` → `lint` → `build` → `wrangler deploy --dry-run` |
| `.dev.vars.example` | create | Names only, **no values** (`FR-192`) |

#### Implementation order

| # | Step | Requirement |
|---|---|---|
| P1.1 | Confirm G0-1/G0-9 `Met` in writing; stop if not | — |
| P1.2 | Extend `.gitignore` **before** creating any file that could contain a secret | `AGENTS.md` §4.3 |
| P1.3 | Initialise `package.json`; add Astro, adapter, TypeScript, ESLint. Resolve exact versions from the registry at this moment — do not copy stale versions from any document | `NFR-014`, `NFR-058` |
| P1.4 | Record each dependency's licence finding (package, version, licence, commercial-use clause, Workers compatibility) in the PR | `AGENTS.md` §8 |
| P1.5 | `astro.config.mjs` with the Cloudflare adapter; confirm `output: 'server'` and the passthrough image service | `FR-180`, `FR-198` |
| P1.6 | `wrangler.jsonc` with pinned `compatibility_date`, `nodejs_compat`, `observability`, `DB` and `MEDIA` bindings | `FR-181`, `FR-196` |
| P1.7 | `wrangler types` → commit the generated types | `AC-061` |
| P1.8 | `src/lib/config/site.ts` and `bindings.ts`; `/api/health` proves fail-fast behaviour | `FR-016`, `FR-193` |
| P1.9 | Tokens, base styles, `BaseLayout`, header/footer, skip link, landmarks | `FR-010`, `FR-017` |
| P1.10 | `404.astro`, `500.astro`, `/api/health` | `FR-005`, `NFR-059` |
| P1.11 | Favicon/app-icon derivation from the icon mark; document the non-square limitation instead of upscaling | PRD §10.3 |
| P1.12 | CI workflow; confirm it fails on a type error and on a lint warning | `NFR-011` |
| P1.13 | `wrangler dev` smoke run; `wrangler deploy --dry-run` (dry-run only — **not** a deploy) | `AC-061` (a)(c) |

#### Acceptance criteria

| AC | Scope in this phase | Status |
|---|---|---|
| `AC-061` | (a) local run, (c) `compatibility_date` + `nodejs_compat`, (d) missing-binding error, (f) dependency compatibility record | **Verified**; (b) and (e) `Deferred` → P6 (require deployment) |
| `AC-007` | `id-ID` / `Asia/Jakarta` single constant; tokens applied; no sticky-overlay focus trap | **Verified** |
| `AC-003` | 404 in Bahasa Indonesia with nav + WhatsApp CTA; no placeholder strings in build output | **Verified** (copy list checked against the built HTML) |
| `AC-073` | Preview isolation | **Deferred** → P6 — requires two provisioned databases |
| `NFR-011`, `NFR-053`, `NFR-057`, `NFR-058` | Strict typecheck, bindings-only config, reproducible build, dependency compatibility | **Verified** by CI |

#### Testing and completion evidence

- `npm run typecheck` — zero errors, `strict`.
- `npm run lint` — zero warnings.
- `npm run build` — succeeds; inspect `dist/` for leaked `.dev.vars`, `.db`, or source maps containing secrets.
- `wrangler dev` — `/` and a deliberately unmatched path return 200 and 404 respectively.
- `wrangler deploy --dry-run` — completes and prints the bundle size.
- CI run link attached to the phase PR.
- Screenshot of the 404 page at 360 px and 1280 px.

#### Risks and blockers

| Risk | Mitigation |
|---|---|
| A dependency pulls a Node-only API and fails at runtime, not at build (`RISK-011`) | Every dependency checked for Workers support before adoption; `nodejs_compat` is not a licence to assume Node |
| Placeholder `database_id` accidentally carried into production config | Placeholder marked with a conspicuous comment; Phase 6 gate re-reads the file line by line |
| `.gitignore` left unextended and `.dev.vars` committed | P1.2 runs before anything else; CI secret scan (`AC-080`) |
| Astro adapter option names drift between majors | Verify each option against the adapter's current docs at implementation time; the PRD's own evidence protocol (§12.6) applies |

#### Exit condition

M1 met: CI green (typecheck, lint, build, dry-run), local dev serves `/` and a 404, no Cloudflare resource touched, `.gitignore` verified to exclude `.dev.vars`/`.wrangler/`/build output. Then P2A, P3 and P4.1 may start **in parallel**.

---

### Phase 2 — Public website

#### Purpose and scope

Build the entire public presentation layer — layout, navigation, homepage blocks, catalog, detail pages, services, articles, contact, privacy, 404 — to the copy, structure and brand rules in PRD §5–§6.

**Split into two halves, because they are not the same kind of work.**

| Half | Content | Depends on |
|---|---|---|
| **2A — Presentation** | Layout, tokens application, header/nav/drawer, footer, breadcrumbs, cards, price-display component, WhatsApp component, empty states, pagination, static page shells, responsive behaviour | Phase 1 only |
| **2B — Data-bound surfaces** | Catalog filtering/pagination, product detail with gallery and specs, category archives, article list/detail, homepage blocks fed by real queries, contact form submission | Phase 4.1–4.3 (schema + repository) and Phase 4.10 (settings) |

**Sequencing deviation:** the PRD assigns public pages to P4 (after the CMS). This plan proposes 2A earlier so design and copy can be reviewed before the admin absorbs effort. `OD-11` records this and requires approval. If rejected, 2A moves after P3 with no change to its content.

#### Prerequisites and dependencies

- Phase 1 exit (M1).
- 2A: none beyond Phase 1.
- 2B: `products`, `product_categories`, `posts`, `post_categories`, `services`, `pages`, `settings`, `media` tables (P4.1); repository interface (P4.3); settings values from the owner (`DEP-001`).
- `OD-09` (owner-supplied content) for anything that renders real business facts.

#### Files and modules

| Path | Action | Purpose |
|---|---|---|
| `src/layouts/PublicLayout.astro` | create | Public-only layout; admin styles never imported here (`RISK-009`) |
| `src/components/public/Header.astro`, `NavDrawer.astro` | create | Focus trap, `Esc`, `aria-expanded`, scroll lock, `aria-current` (`FR-009`, `FR-011`) |
| `src/components/public/Footer.astro` | create | Identity, contact block, quick links, privacy link, computed year (`FR-002`) |
| `src/components/public/Breadcrumbs.astro` | create | Visible breadcrumb + link validity (`FR-008`) |
| `src/components/public/{Hero,TrustStrip,ServiceCard,ProductCard,ArticleCard,CtaBand}.astro` | create | Homepage blocks B1–B6 (`FR-020`–`FR-025`) |
| `src/components/public/PriceDisplay.astro` | create | Branches on `price_display`; never empty, never `Rp 0` (`FR-046`) |
| `src/components/public/WhatsAppCta.astro` | create | Single place that builds `wa.me` URLs (`FR-007`, `FR-047`) |
| `src/components/public/{EmptyState,Pagination,FilterBar}.astro` | create | `FR-044`, `FR-040`, `FR-041` |
| `src/pages/index.astro` | modify | Six blocks, each omitted entirely when it has no eligible content (`FR-039`) |
| `src/pages/tentang-kami.astro` | create | CMS body (`FR-026`, `FR-031`) |
| `src/pages/layanan/index.astro`, `src/pages/layanan/[slug].astro` | create | List + detail with process steps (`FR-027`, `FR-037`) |
| `src/pages/produk/index.astro`, `produk/[slug].astro`, `produk/kategori/[slug].astro` | create | Catalog, detail, category archive (`FR-040`–`FR-054`) |
| `src/pages/blog/index.astro`, `blog/[slug].astro`, `blog/kategori/[slug].astro` | create | `FR-060`–`FR-070` |
| `src/pages/kontak.astro` | create | Contact block, hours, map-if-valid, form (`FR-028`, `FR-032`, `FR-036`) |
| `src/pages/kebijakan-privasi.astro` | create | CMS body (`FR-030`) |
| `src/lib/format/{currency,datetime,whatsapp}.ts` | create | Pure, unit-tested; integer rupiah, `Asia/Jakarta`, `0`→`62` normalisation (`FR-122`) |
| `src/lib/pricing/display.ts` | create | Pure price-branching function (`FR-046`, `AC-018`, `AC-068`) |
| `src/lib/repositories/types.ts` | create | The contract P2B and P4.3 share |
| `src/lib/repositories/fixtures/*.ts` | create | Dev-only fixtures, **no fabricated business facts** (`FR-038`); gated to non-production |
| `src/pages/api/kontak.ts` | create | Server-validated, throttled, hashed IP (`FR-029`, `FR-035`) |

#### Implementation order

| # | Step | Requirement |
|---|---|---|
| P2.1 | `PublicLayout` + header + drawer + footer + skip link; verify at 360 px first | `FR-001`, `FR-002`, `FR-009`, `FR-010` |
| P2.2 | `src/lib/format/*` and `pricing/display.ts` with unit tests **before** any component uses them | `FR-046`, `FR-122` |
| P2.3 | `PriceDisplay` + `WhatsAppCta` components, driven by injected values | `FR-046`, `FR-047` |
| P2.4 | Static page shells: `tentang-kami`, `kebijakan-privasi`, `layanan` list | `FR-026`, `FR-030`, `FR-027` |
| P2.5 | Homepage block composition with omission logic; prove omission with empty fixtures | `FR-039`, `AC-008` |
| P2.6 | 404 already exists — verify it keeps the shell and CTA after the layout lands | `FR-005` |
| P2.7 | *(after P4.3)* Catalog: server-side pagination, category/brand/search/sort in the URL, deterministic ordering | `FR-040`–`FR-043`, `FR-052`, `FR-059` |
| P2.8 | *(after P4.3)* Product detail: gallery order, specs table, description, related products, inquiry CTA | `FR-045`, `FR-048`–`FR-050`, `FR-057` |
| P2.9 | *(after P4.3)* Article list/detail/category: pagination, author, Jakarta-formatted date, related, TOC | `FR-060`–`FR-070` |
| P2.10 | *(after P4.3)* Category archives, both content types, published-only | `FR-054`, `FR-069` |
| P2.11 | *(after P4.10)* Contact page driven by settings; form POST with validation, throttle, hashed IP | `FR-028`, `FR-029`, `FR-035`, `NFR-020` |
| P2.12 | Responsive pass 320/360/430/768/1024/1280/1536; no horizontal scroll | `NFR-004` |

#### Acceptance criteria

| AC | Scope | Status |
|---|---|---|
| `AC-001` | Drawer behaviour, footer contact block, `aria-current`, `rel="noopener noreferrer"` | **Verified** (drawer/footer) |
| `AC-002` | `lang="id"`, Bahasa Indonesia only, main heading present without JS | **Verified** via `curl` on each route |
| `AC-003` | 404 + no placeholder strings | **Verified** |
| `AC-004` | Empty category absent from nav; breadcrumbs resolve | **Verified** with fixtures; re-verified against D1 in P4 |
| `AC-005` | WhatsApp URL uses normalised CMS number | **Deferred** → P4 exit (needs settings) |
| `AC-006` | Skip link, landmarks, one `h1`, heading order | **Verified** |
| `AC-008` | Homepage blocks reflect CMS values without redeploy | **Deferred** → P4 exit |
| `AC-010`, `AC-012`, `AC-013`, `AC-014` | CMS-editable bodies, contact details, form behaviour | **Deferred** → P4 exit |
| `AC-011`, `AC-015`–`AC-019`, `AC-021`–`AC-025` | Content surfaces, filters, price modes, inquiry CTA | **Verified** against fixtures; **re-run against D1** in P5 |
| `AC-020` | Product SEO fields, draft 404, JSON-LD, alt-text gate | **Partial** — draft-404 and alt gate `Deferred` → P4; SEO half → P3 |
| `NFR-004`, `NFR-020` | No horizontal scroll; data minimisation | **Verified** |

#### Testing and completion evidence

- `curl` each public URL and confirm the main heading text is in the HTML with JavaScript disabled.
- Unit tests: price branching (all four modes + missing amount), WhatsApp normalisation (`0812-…` → `62812…`), rupiah formatting, Jakarta date rendering.
- Fixture-driven tests: 13 products at 12/page → 12 + 1; empty catalog → empty state with CTA; empty homepage blocks → zero containers in output.
- Responsive screenshots at the seven widths; keyboard-only pass on the drawer.
- Owner review sign-off on copy and layout (M2).

#### Risks and blockers

| Risk | Mitigation |
|---|---|
| Fixtures leak into production and publish invented business facts (`RISK-006`) | Fixture adapter refuses to load when the runtime is not local; CI greps the build for fixture markers |
| Components bind directly to D1, defeating the contract | Repository interface only; no `env.DB` import inside `src/components/**` |
| Admin CSS leaks into public bundles (`RISK-009`) | Separate layout trees; CI inspects the public bundle for admin class prefixes |
| Empty-state logic regresses into placeholder copy | `FR-039` test asserts absence, not presence |
| `OD-11` rejected after 2A is built | 2A is ordered so it is reusable in the PRD's original sequence — no rework beyond scheduling |

#### Exit condition

M2: every route in PRD §5.1 exists and renders; a11y checklist passes; owner has reviewed design and copy; no placeholder or fabricated claim in the build; 2B items either complete (if Phase 4.1–4.3 landed) or explicitly parked with the ACs above marked `Deferred`.

---

### Phase 3 — Content & SEO

#### Purpose and scope

Make the public site machine-readable and honest: one canonical per page, unique metadata, OG/Twitter, a generated sitemap that contains only indexable URLs, a `robots.txt` that excludes `/admin` and `/api`, JSON-LD emitted only when the data supports it, and image delivery that never transforms pixels on the server.

#### Prerequisites and dependencies

- Phase 1 exit; `BaseLayout` stable (end of P2.1).
- `OD-04` (media hostname) for absolute media URLs.
- Phase 4.11 for slug-history-driven 301s (`AC-054`/`AC-071`).
- Phase 4.13 for preview `noindex` (`AC-053`).

#### Files and modules

| Path | Action | Purpose |
|---|---|---|
| `src/components/seo/SeoHead.astro` | create | Title template, description, canonical, OG, Twitter, verification token (`FR-140`–`FR-142`, `FR-157`) |
| `src/components/seo/JsonLd.astro` | create | Serialises a validated object; emits nothing when the object is `null` |
| `src/lib/seo/canonical.ts` | create | Absolute URL from the configured origin; trailing-slash policy (`FR-141`, `FR-154`) |
| `src/lib/seo/metadata.ts` | create | Per-page title/description resolution with documented defaults (`FR-140`, `FR-129`) |
| `src/lib/seo/structured-data.ts` | create | `Organization`/`LocalBusiness` (only with complete data), `Product` (offers only when priced), `BreadcrumbList`, `BlogPosting` (`FR-145`–`FR-148`) |
| `src/lib/seo/sitemap.ts` | create | Indexable-only URL set + honest `lastmod` (`FR-143`, `FR-156`) |
| `src/pages/sitemap.xml.ts` | create | On-demand generated XML (a build-time integration cannot see D1 changes) |
| `src/pages/robots.txt.ts` | create | Disallows `/admin`, `/api`, preview paths; absolute sitemap reference (`FR-144`) |
| `src/components/public/ResponsiveImage.astro` | create | `srcset`/`sizes`, explicit width/height, lazy below fold, pre-generated variant keys only (`FR-106`, `FR-113`, `NFR-019`) |
| `src/middleware.ts` | create | Trailing-slash and lowercase normalisation redirects; preview `X-Robots-Tag` (`FR-012`, `FR-149`, `FR-154`) |
| `src/lib/seo/redirects.ts` | create | Slug-history lookup, chain collapsing, loop refusal (`FR-150`, `FR-177`) |

#### Implementation order

| # | Step | Requirement |
|---|---|---|
| P3.1 | `canonical.ts` + `metadata.ts` + `SeoHead` wired into `BaseLayout`; assert exactly one canonical in output | `FR-141`, `AC-049` |
| P3.2 | OG/Twitter tags + default OG image fallback | `FR-142`, `FR-153` |
| P3.3 | `structured-data.ts` with a "null means omit" contract; unit-test the omission path first | `FR-145`, `FR-052` |
| P3.4 | `sitemap.xml` endpoint: published only, no `/admin`, no `/api`, absolute URLs | `FR-143`, `AC-051` |
| P3.5 | `robots.txt` endpoint | `FR-144` |
| P3.6 | `ResponsiveImage` bound to the agreed variant spec (`OD-07`, which is the G0-6 detail) | `FR-106`, `FR-113` |
| P3.7 | *(after P4.11)* 301 handler from `slug_history`, chain collapsing, loop refusal | `FR-150`, `FR-177`, `AC-054` |
| P3.8 | *(after P4.13)* preview `noindex` header + meta; sitemap exclusion | `FR-149`, `AC-053` |
| P3.9 | Alt-text audit query surfacing content images without alt text | `FR-151` |
| P3.10 | Copy audit: grep the build for ranking/position/traffic/guarantee claims | `FR-152`, `AC-079` |

#### Acceptance criteria

| AC | Scope | Status |
|---|---|---|
| `AC-049` | Unique title/description, one absolute canonical matching the served URL | **Verified locally**; absolute-origin part `Deferred` → P6 (needs the live hostname) |
| `AC-050` | OG/Twitter set, default image fallback | **Verified** |
| `AC-051` | Sitemap valid, robots correct, verification token renders | **Verified** |
| `AC-052` | JSON-LD valid; omitted when data is incomplete; `offers` only when priced | **Verified** in a validator, with and without the optional data |
| `AC-053`, `AC-054`, `AC-070`, `AC-071` | Preview noindex; slug 301; chain collapse; slug-reuse block | **Deferred** → P4 exit (need Phase 4.11/4.13) |
| `AC-055` | Alt-text audit, no ranking claims, 404 not 200, no IDs in URLs, self-canonical pagination — including `FR-158` human-readable slugs, verified by route review | **Verified** |
| `AC-060` (a11y + `srcset` parts) | Accessibility score, explicit dimensions, lazy loading | **Partial**; full measurement → P5 |

#### Testing and completion evidence

- XML validation of `/sitemap.xml`; assert zero draft URLs and zero `/admin` entries.
- JSON-LD validated in a schema validator, twice: complete data and one required field removed.
- `curl -I` each route: one `rel="canonical"`, absolute, matching the request.
- Grep the built HTML for forbidden claim strings.
- Before/after HTML diff proving the omission path renders no empty script tag.

#### Risks and blockers

| Risk | Mitigation |
|---|---|
| A build-time sitemap integration silently caches and ships stale URLs | On-demand endpoint; test after a data change |
| JSON-LD emitted with placeholder values (`RISK-006`) | `null`-means-omit contract; unit test asserts omission |
| Canonical points at a redirect or 404 | Test canonical of every route against its own status code |
| Sitemap includes a draft via a status filter bug | Explicit `status = 'published'` in the query plus a negative test |

#### Exit condition

M3: canonical/metadata/OG verified on every indexable route; sitemap and robots validated; JSON-LD validated in both data states; image component emits `srcset` with explicit dimensions and performs no transformation. `AC-053`/`AC-054` remain `Deferred` and are carried into the Phase 4 exit checklist.

---

### Phase 4 — Admin & data

#### Purpose and scope

Deliver the durable data layer and the CMS the owner actually operates: D1 schema and migrations, prepared-SQL repositories with `batch()` atomicity, authentication and sessions, media upload to R2 through an adapter, settings/branding, the publishing state machine, slug history, preview tokens, audit logging, and structured observability.

This is the largest phase and the one with the most security surface.

#### Prerequisites and dependencies

- Phase 1 exit.
- G0-4 (D1 IDs) for **remote** work only — local emulation needs none.
- G0-5 + `OD-04` for media acceptance.
- G0-7 + `OD-03` for password hashing — **hard blocker on P4.5**.
- `OD-05` (rich text editor + storage format) — **hard blocker on P4.7/P4.8 body editing**.
- `OD-06` (admin island framework) — **hard blocker on P4.6**.
- `OD-08` (purge mechanism) for `FR-125`/`FR-179`.
- `OD-14` (spam protection) for the contact form.

#### Files and modules

| Path | Action | Purpose |
|---|---|---|
| `migrations/0001_init.sql` … `000N_*.sql` | create | Schema per PRD §13.5; append-only once applied anywhere |
| `migrations/README.md` | create | Apply order, environment order, rollback stance |
| `src/lib/db/client.ts` | create | `env.DB` accessor; no connection string, no `file:` (`FR-183`) |
| `src/lib/db/batch.ts` | create | Single-`batch()` helper; no `BEGIN` across awaits (`FR-185`) |
| `src/lib/db/queries/{products,posts,services,pages,media,settings,sessions,slug-history,audit,inquiries}.ts` | create | Prepared statements only (`OD-16`/`OQ-020`) |
| `src/lib/repositories/d1/*.ts` | create | Implements the Phase 2 contract |
| `src/lib/auth/password.ts` | create | PBKDF2-SHA256 via `crypto.subtle`; per-password salt; parameters recorded with the hash |
| `src/lib/auth/session.ts` | create | D1-backed sessions; HMAC-signed `HttpOnly` cookie; absolute + idle expiry (`FR-194`, `FR-083`) |
| `src/lib/auth/csrf.ts` | create | Origin/`Sec-Fetch-Site` check + per-session token (`FR-084`) |
| `src/lib/auth/throttle.ts` | create | 5 failures / 15 min / IP+account, D1 counters, generic error (`FR-085`) |
| `src/lib/validation/*.ts` | create | Boundary validators with field-level Bahasa Indonesia messages (`FR-169`) |
| `src/lib/sanitize/html.ts` | create | Allowlist sanitiser applied on save **and** on render (`AGENTS.md` §5) |
| `src/lib/media/adapter.ts` | create | `putObject` / `deleteObject` / `getPublicUrl` / `replaceObject` (`FR-101`) |
| `src/lib/media/r2-adapter.ts`, `memory-adapter.ts` | create | R2 implementation + in-memory test double |
| `src/lib/media/validate.ts` | create | Magic bytes, size, format allowlist, SVG rejection (`FR-103`, `FR-104`, `FR-114`) |
| `src/lib/media/exif.ts` | create | Strip EXIF/GPS for formats where it is **verified**; reject the format otherwise |
| `src/lib/publishing/transitions.ts` | create | Legal state machine, server-side enforcement (`FR-160`, `FR-170`) |
| `src/lib/publishing/gate.ts` | create | Publish validation (`FR-162`) |
| `src/lib/cache/purge.ts` | create | Purge adapter: Cloudflare API implementation + local no-op (`FR-125`, `FR-179`, `FR-186`) |
| `src/lib/preview/token.ts` | create | Short-lived signed preview tokens (`FR-161`) |
| `src/lib/audit/log.ts` | create | Actor, action, entity, UTC timestamp, no secrets/PII (`FR-093`) |
| `src/pages/admin/masuk.astro` | create | Login (`FR-080`, `FR-098`) |
| `src/pages/admin/index.astro` | create | Dashboard counts + last-updated list (`FR-087`, `FR-088`) |
| `src/pages/admin/produk/{index,baru,[id],kategori}.astro` | create | Product management (`FR-040`–`FR-058` admin side) |
| `src/pages/admin/blog/{index,baru,[id],kategori}.astro` | create | Article management |
| `src/pages/admin/media/index.astro` | create | Media library (`FR-108`–`FR-118`) |
| `src/pages/admin/pengaturan/{index,kontak,branding,seo}.astro` | create | Settings groups (`FR-120`–`FR-139`) |
| `src/pages/admin/keluar.ts` | create | Server-side session invalidation (`FR-086`) |
| `src/pages/api/admin/**` | create | Each route independently re-validates the session (`FR-082`, `FR-092`) |
| `src/pages/api/media/upload.ts` | create | Streamed upload; no filesystem write (`FR-182`, `FR-187`) |
| `src/pages/api/preview/[...token].ts` | create | Preview render (`FR-161`) |
| `src/middleware.ts` | modify | UX redirect only — never the authorisation boundary |
| `src/lib/observability/log.ts` | create | Structured logs with a correlation ID (`FR-196`, `NFR-055`) |

#### Implementation order

| # | Step | Requirement |
|---|---|---|
| P4.1 | Schema migrations for all 13 tables in PRD §13.5 + indexes for every public listing query | `FR-184`, `NFR-046` |
| P4.2 | Apply to **local** D1; run twice to prove idempotency | `AC-062` (local half) |
| P4.3 | `db/client.ts`, `batch.ts`, queries, D1 repositories implementing the Phase 2 contract | `FR-183`, `FR-185` |
| P4.4 | PBKDF2 benchmark harness on local `workerd`; record iterations, p50/p95 CPU, and the tier budget; owner approves parameters | `AC-027`, `NFR-042`, `OD-03` |
| P4.5 | `password.ts`, `session.ts`, `csrf.ts`, `throttle.ts`; unit + integration tests | `AC-026`–`AC-032`, `AC-067` |
| P4.6 | Admin shell: sidebar, top bar, breadcrumb, 360 px usable; isolated CSS | `FR-090`, `RISK-009` |
| P4.7 | Products CRUD incl. specs editor, price modes, images with alt text, validation | `FR-169`, `FR-162` |
| P4.8 | Articles CRUD incl. editor integration per `OD-05` | `FR-070`, `FR-168` |
| P4.9 | Media: validate → strip EXIF → store via adapter → insert row; delete object **before** row | `FR-103`–`FR-112`, `FR-191` |
| P4.10 | Settings groups + validation + last-saved timestamp; propagation to public pages | `FR-120`–`FR-139` |
| P4.11 | Publishing state machine, slug history, redirect resolution, slug-reuse block | `FR-160`–`FR-179` |
| P4.12 | Audit log entries inside the same `batch()` as the content write | `FR-093`, `AC-035` |
| P4.13 | Preview tokens + `X-Robots-Tag` | `FR-161`, `FR-149` |
| P4.14 | Cache purge on publish/unpublish/settings; admin responses `no-store` | `FR-125`, `FR-179`, `FR-186` |
| P4.15 | Structured logging for auth, publish, upload failure, unhandled error | `FR-196` |
| P4.16 | Admin route `noindex` + `robots.txt` disallow confirmation | `FR-097` |

#### Acceptance criteria

| AC | Scope | Status |
|---|---|---|
| `AC-026`–`AC-032`, `AC-034` | Login, hashing, session, CSRF, throttle, expiry, single role | **Verified** |
| `AC-067` | `curl` admin routes with no cookie → redirect/401, no data | **Verified** |
| `AC-062` | Migration from empty + idempotent re-run | **Verified locally**; remote half `Deferred` → P6 |
| `AC-063` | Data survives redeploy | **Deferred** → P6 (needs a deployment) |
| `AC-033`, `AC-035`, `AC-036` | Dashboard, audit trail, admin noindex + Bahasa Indonesia | **Verified** |
| `AC-037`–`AC-043` | Upload validation, adapter boundary, opaque keys, EXIF, variants, library, deletion, replacement | **Verified locally** with the memory adapter + emulated R2 |
| `AC-044`–`AC-046`, `AC-048` | Settings propagation, branding validation, no secrets in settings | **Verified** |
| `AC-047` | Purge on save | **Deferred** → P5 (verify) / P6 (real edge cache) |
| `AC-053`, `AC-054`, `AC-070`, `AC-071` | Preview noindex, slug 301, chain collapse, slug reuse | **Verified** |
| `AC-056`, `AC-057` | Illegal transitions rejected, publish gate, unpublish/archive | **Verified** |
| `AC-064` | R2 URL on the custom domain, survives redeploy | **Deferred** → P6 |
| `AC-069` | `batch()` atomicity with fault injection | **Verified** |
| `AC-075` | Deletion order + orphan reconciliation | **Verified locally**; scheduled job → P6 |
| `AC-080` | No secret in the client bundle | **Verified** by scan |
| `AC-008`, `AC-010`, `AC-012`, `AC-013`, `AC-045` | Carried over from Phase 2 as `Deferred` | **Verified** here |

#### Testing and completion evidence

- Migration test: empty database → apply → re-apply → `d1_migrations` unchanged.
- Fault-injection test: third statement of a publish batch forced to fail → all three tables show the pre-operation state.
- `curl` without a cookie for every admin route and admin API.
- Stale-cookie test: delete the session row, replay the cookie, expect rejection.
- CSRF test: state-changing request with a foreign `Origin` → 403, no mutation.
- Upload matrix: valid JPEG, HTML bytes named `.png`, oversize, SVG → one success, three rejections, zero partial objects.
- Password benchmark record attached to the decision.
- Log sample review: no password, no token, no full PII.
- Owner walkthrough of J5 and J6 (PRD §3.4) on the local environment.

#### Risks and blockers

| Risk | Mitigation |
|---|---|
| A multi-table write implemented as separate awaits (`RISK-012`) | One `batch()` per operation; code-review checklist item; `AC-069` in CI |
| CPU budget exceeded by hashing or rich-text parsing (`RISK-011`) | Benchmark first (P4.4); re-measure the worst request path; keep work off the request path |
| Filesystem used for uploads (`RISK-004`) | Adapter only; grep for `node:fs` in CI |
| EXIF stripping claimed but untested for a format | Strip only what is verified; reject the rest |
| Admin CSS/JS leaking to public pages (`RISK-009`) | Separate layout tree; bundle inspection in CI |
| Purge API token stored as a CMS setting | Token as a Worker secret only (`FR-127`) |
| Slug history written outside the transaction | Included in the same `batch()` as the slug update |

#### Exit condition

M4: the owner completes J5 (create → preview → publish a product) and J6 (rename a published slug → 301) on the local environment; all ACs marked `Verified` above pass; the deferred set is listed in §7.1 with the phase that will close it; no secret, `.db`, or generated media is staged.

---

### Phase 5 — Testing & hardening

#### Purpose and scope

Turn the accumulated manual verification into a repeatable suite, then close the performance, security-header and resilience targets on a real preview deployment. Nothing new is built here except tests and hardening.

#### Prerequisites and dependencies

- Phase 4 exit.
- Preview deployment available → G0-2…G0-5, G0-8 (`OD-02`, `OD-07`).
- A decision on the test toolchain (`OD-13`) and on `OD-08` for purge measurement.

#### Files and modules

| Path | Action | Purpose |
|---|---|---|
| `vitest.config.ts`, `tests/setup.ts` | create | Test runner; Workers-runtime pool for anything touching bindings |
| `tests/unit/**` | create | Slug normalisation, redirect chain collapsing, price branching, settings validation, WhatsApp normalisation |
| `tests/integration/**` | create | Publish gate, slug-history writes, `batch()` fault injection, migrations from empty, adapter contract, magic-byte/size rejection |
| `tests/e2e/**` | create | Login/logout, unauthenticated admin via `curl`, draft→preview→publish, unpublish, slug 301, four price modes, upload → URL survives |
| `scripts/secret-scan.*` | create | Fails on secret patterns in `dist/` (`AC-080`) |
| `scripts/check-claims.*` | create | Fails on ranking/certification/testimonial patterns (`AC-079`) |
| `src/middleware.ts` | modify | CSP report-only → enforce; `nosniff`, `Referrer-Policy`, frame protection (`NFR-013`) |
| `docs/qa/RESPONSIVE.md`, `docs/qa/ACCESSIBILITY.md` | create | Recorded manual pass evidence |
| `docs/qa/PERFORMANCE.md` | create | Measured numbers with tier, date and parameters (`AC-076`) |

#### Implementation order

| # | Step | Requirement |
|---|---|---|
| P5.1 | Test runner + first 20 unit tests over pure modules | `AGENTS.md` §9 |
| P5.2 | Integration suite against local D1/R2, including fault injection | `AC-069`, `AC-062` |
| P5.3 | E2E suite via `curl` for auth boundaries (no UI dependency) | `AC-067` |
| P5.4 | Security headers: report-only first, review reports, then enforce | `NFR-013` |
| P5.5 | Secret scan + claim scan wired into CI | `AC-080`, `AC-079` |
| P5.6 | Lighthouse mobile on the preview deployment; record LCP/CLS/INP | `AC-060` |
| P5.7 | CPU profile per request path; redesign any path above 50 % of budget | `NFR-042`, `AC-076` |
| P5.8 | Cache correctness: draft never served from cache; edit visible within 30 s | `AC-070`, `AC-072` |
| P5.9 | Resilience: simulated D1 failure → generic Bahasa Indonesia error, logged | `NFR-059`, `AC-078` |
| P5.10 | Manual responsive matrix + keyboard + screen-reader spot check + contrast | `NFR-003`, `NFR-006` |

#### Acceptance criteria

| AC | Status |
|---|---|
| `AC-060` | **Verified** on preview (the only place it can be measured) |
| `AC-072`, `AC-076`, `AC-078` | **Verified** on preview |
| `AC-069`, `AC-066`, `AC-037`, `AC-042` | **Verified** in CI, not only by hand |
| `AC-079`, `AC-080` | **Verified** by automated scan |
| `NFR-003`, `NFR-006`, `NFR-013`, `NFR-021` | **Verified** |
| `AC-077` | **Deferred** → P6 (cost model is a launch artifact) |

#### Testing and completion evidence

- CI run: typecheck, lint, unit, integration, e2e, secret scan, claim scan — all green.
- Lighthouse report JSON committed under `docs/qa/`.
- CPU profile table per route with the tier's budget stated.
- Cache-status header evidence before and after a purge.
- Manual QA docs filled in with widths, browsers, and findings.

#### Risks and blockers

| Risk | Mitigation |
|---|---|
| Targets measured on Free tier, then re-measured on Paid with different results | Record tier and date with every number; `AC-076` requires it |
| Tests pass locally but not under the Workers runtime | Integration tests run in the Workers pool, not Node |
| Hardening pass turns into a refactor | Scope to the ACs listed; anything else is a separate change |
| CSP enforcement breaks the admin island | Report-only first, review violations, then enforce |

#### Exit condition

M5: full CI green including e2e and scans; `AC-060`/`AC-072`/`AC-076`/`AC-078` measured and recorded with parameters; CSP enforced; every deferred AC from Phases 2–4 that does not require production is closed.

---

### Phase 6 — Deployment readiness

#### Purpose and scope

Assemble everything needed for a launch that **someone else** will perform, and prove the recovery path works. This phase produces configuration, checklists, scripts and records. **It performs no deployment, creates no Cloudflare resource, and changes no DNS.**

#### Prerequisites and dependencies

- Phase 5 exit.
- G0-2…G0-5 `Met` with recorded IDs (account, zone, `gjdb` production + preview, `gj-cctv` + media hostname).
- G0-8 with a D1 primary location decision.
- `OD-04` (media hostname), `OD-08` (purge credential), `OD-10` (privacy wording), `OD-15` (content).

#### Files and modules

| Path | Action | Purpose |
|---|---|---|
| `wrangler.jsonc` | modify | `env.preview` and `env.production` blocks; separate `database_id` and bucket per environment (`FR-195`) |
| `docs/runbooks/DEPLOY.md` | create | Ordered, copy-pasteable deploy procedure including the migration step (`FR-199`) |
| `docs/runbooks/SECRETS.md` | create | Which secret, set with `wrangler secret put`, by whom — **names only, no values** (`FR-192`) |
| `docs/runbooks/DNS.md` | create | Record types, names, targets, and who applies them. **Documentation only** |
| `docs/runbooks/BACKUP_RESTORE.md` | create | Export cadence, retention, restore procedure, drill record (`FR-197`, `NFR-056`) |
| `docs/runbooks/ROLLBACK.md` | create | Deployment rollback and its limits (does not roll back data) |
| `src/workers/scheduled.ts` or adapter-configured entry | create | Scheduled D1 export → R2. Confirm the adapter's supported mechanism for a custom Worker entry point at implementation time |
| `docs/COST_MODEL.md` | create | Requests, CPU, D1 rows/storage, R2 storage/operations, transformation counts, free-tier ceiling, 70 % revisit trigger (`AC-077`) |
| `docs/qa/SMOKE_TEST.md` | create | Post-deploy smoke checks with expected status codes |
| `docs/qa/RESTORE_DRILL.md` | create | Date, operator, source dump, target database, row-count comparison (`AC-074`) |

#### Implementation order

| # | Step | Requirement |
|---|---|---|
| P6.1 | Write the environment matrix: local / preview / production, each with its own D1 ID and bucket | `FR-195` |
| P6.2 | Fill `wrangler.jsonc` env blocks from the owner-supplied IDs; re-read every line for a stray placeholder | `FR-199` |
| P6.3 | Secrets inventory; owner performs `wrangler secret put` | `FR-192` |
| P6.4 | Backup: implement the scheduled export, then **perform a restore into a non-production database and record it** | `FR-197`, `AC-074` |
| P6.5 | Orphan reconciliation job (R2 keys vs `media.storage_key`), both directions | `FR-191` |
| P6.6 | DNS plan document: `cctv.artupski.com` → Worker route; media hostname → R2 custom domain | `FR-188` |
| P6.7 | Cost model with the free-tier ceiling stated | `AC-077` |
| P6.8 | Smoke-test checklist covering every public route, the admin login, one publish, one upload, sitemap, robots | — |
| P6.9 | Final gate audit: every `Deferred` AC from §7.1 closed or explicitly accepted by the owner | — |

#### Acceptance criteria

| AC | Status |
|---|---|
| `AC-058`, `AC-059`, `AC-062` (remote), `AC-063`, `AC-064`, `AC-073` | **Verified** on preview — cross-environment query must fail |
| `AC-047`, `AC-072` | **Verified** with a real cache purge |
| `AC-049` | **Verified** with the production origin |
| `AC-074` | **Verified** by a recorded restore drill |
| `AC-077` | **Verified** by the written cost model |
| `AC-061` (b)(e) | **Verified** — deployment runs on Workers, reproducible from the repo |
| `AC-078` | **Verified** — observability queryable in the real environment |

#### Testing and completion evidence

- Preview environment: cross-environment query attempt fails; a preview record is absent from production.
- Restore drill record with matching row counts.
- Media URL returns 200 from the custom domain, not `r2.dev`.
- Purge timing measured under 30 s.
- Smoke-test checklist fully ticked **on preview**, not on production.
- Cost model reviewed against actual preview usage.

#### Risks and blockers

| Risk | Mitigation |
|---|---|
| A preview `database_id` equal to production | P6.2 line-by-line read; `AC-073` test |
| Backup scheduled but never restored (`NFR-056`) | The drill is the acceptance criterion, not the schedule |
| Media served from `r2.dev` in production (`RISK-013`) | Custom domain is a hard requirement; grep content for `r2.dev` |
| Deployment performed as part of "readiness" | Explicit boundary: this plan stops before deploy |
| Secret pasted into `docs/` while writing the runbook | Runbooks contain names only; secret scan covers `docs/` too |

#### Exit condition

M6: every runbook written; environment matrix complete with real IDs; restore drill recorded; cost model written; smoke-test script ready; **no resource created, no DNS changed, nothing deployed**. Launch itself is a separate, owner-authorised action.

---

### 7.1 Deferred acceptance criteria register

| AC | Deferred from | Closes in | Why it cannot close earlier |
|---|---|---|---|
| `AC-005`, `AC-008`, `AC-010`, `AC-012`, `AC-013`, `AC-045` | Phase 2 | Phase 4 exit | Need settings + D1 |
| `AC-020` (draft-404, alt gate) | Phase 2 | Phase 4 exit | Needs the publishing gate |
| `AC-047`, `AC-072` | Phase 3/4 | Phase 5 (verify) / Phase 6 (real cache) | Need a real edge cache |
| `AC-049` (absolute origin) | Phase 3 | Phase 6 | Needs the production hostname |
| `AC-053`, `AC-054`, `AC-070`, `AC-071` | Phase 3 | Phase 4 exit | Need Phase 4.11/4.13 |
| `AC-058`, `AC-059`, `AC-063`, `AC-064` | Phases 1–4 | Phase 6 | Explicitly require a deployment and a custom domain |
| `AC-061` (b)(e) | Phase 1 | Phase 6 | Require a deploy |
| `AC-062` (remote half) | Phase 4 | Phase 6 | Requires the provisioned database |
| `AC-073` | Phase 1 | Phase 6 | Requires two provisioned environments |
| `AC-074`, `AC-077` | — | Phase 6 | Launch artifacts |

---

## 8. Genuinely blocking open decisions

These are the items that stop work. Each is phrased so the owner can answer it in one line.

| ID | Decision needed | Blocks | Impact if unanswered | Recommended default |
|---|---|---|---|---|
| `OD-01` | Approve `PRD.md` v0.2.2 as a document (G0-9) | **Everything** | No phase can start | Approve, or state what must change first |
| `OD-02` | Supply account ID, zone ID, `gjdb` prod + preview `database_id`, `gj-cctv` bucket, media hostname (G0-2…G0-5) | Phase 4 remote work, Phase 6 | Development can continue locally; nothing can be verified or deployed | Owner supplies via the `verify-*.ps1` scripts once re-attached |
| `OD-03` | **Circular gate on G0-7.** The benchmark must run "on the real runtime", which requires a scaffold, which the gate forbids until G0-7 is met. Choose: (a) permit a throwaway, uncommitted benchmark harness as a named gate exception; or (b) accept PBKDF2-SHA256 provisionally with the benchmark as a Phase 1 exit gate and a hard stop before P4.5 | Phase 4 auth (`AC-027`) | Auth cannot be implemented to a fixed spec | (b) — smaller exception, still measured before auth code exists |
| `OD-04` | Media hostname for `gj-cctv` (e.g. `media.cctv.artupski.com`) and confirmation that the zone is in the same account (`D-24`, `OQ-017`) | `FR-188`, `AC-064`, Phase 3 image URLs, Phase 6 | Media cannot be delivered correctly; `r2.dev` would be a release blocker | Owner names the hostname; agent does not create it |
| `OD-05` | Rich text editor and storage format (`OQ-005`, `D-11`, `D-12`) | Phase 4.7/4.8 body editing | Products and articles cannot be authored beyond plain text | Sanitised HTML with a strict allowlist, re-sanitised on render; choose the editor after the §8 licence gate |
| `OD-06` | Admin island framework (React / Preact / Svelte / vanilla) | Phase 4.6 | Admin shell cannot be built | The smallest runtime that passes the licence gate; vanilla + minimal islands if the editor allows |
| `OD-07` | Image variant specification: widths, formats, max upload dimension, who generates them (G0-6 detail) | Phase 2 image component, Phase 4 upload | `FR-106`/`FR-115` unimplementable | 480/768/1024/1600 px WebP + original; client-side downscale before upload |
| `OD-08` | Purge mechanism and credential: Cache Purge API token vs Cache Tags, and who holds it | `FR-125`, `FR-179`, `AC-072` | Publishing cannot invalidate the edge | API token as a Worker secret, zone-scoped, held by the owner |
| `OD-09` | Owner-supplied business facts: address, phone, WhatsApp, hours, service area, real product data, privacy wording (`DEP-001`, `OQ-008`, `OQ-010`) | `AC-009`, `AC-010`, `AC-012`, `AC-079`, launch | Site launches empty or, worse, with invented claims | Publish only confirmed data; omit the rest |
| `OD-10` | Contact form enabled or WhatsApp-only (`OQ-010`), and spam protection (`OD-14`) | `FR-029`, Phase 2.11 | Contact page scope undefined | Enabled with Turnstile **only if the owner approves the service**; otherwise a honeypot + D1 throttle |
| `OD-11` | Approve or reject the sequencing deviation: public presentation before admin (Phase 2 vs PRD §18's P4) | Phase 2 start order | No impact on content, only on schedule | Approve 2A early; keep 2B after P4.3 |
| `OD-12` | Permission to extend the existing one-line `.gitignore` | Phase 1 step P1.2 | Secrets could be committed | Approve; it is the first file touched |
| `OD-13` | Test toolchain: Vitest + Workers pool, and whether Playwright is in scope | Phase 5 | Verification stays manual | Vitest + `@cloudflare/vitest-pool-workers`; `curl`-based e2e; Playwright optional |
| `OD-14` | Spam protection mechanism for the contact form | `FR-029` | Public form is abusable | Honeypot + D1 rate limit; Turnstile only with owner approval |
| `OD-15` | Content-entry window and reviewer (`DEP-010`) | M6, launch | Launch slips or ships thin | Schedule before P6 |
| `OD-16` | ORM confirmation: `D-07` says "not yet chosen" while `OQ-020` says prepared SQL. Reconcile the row | Phase 4.3 | Ambiguous instruction to the implementer | Plain prepared statements + `batch()` per `OQ-020` |

### 8.1 Non-blocking observations

| ID | Observation | Action |
|---|---|---|
| `OBS-01` | `PRD.md` §13.4 rows `D-07`, `D-19`, `D-21`, `D-22`, `D-23` have status **Confirmed** but their "Leading option" cells still list the alternatives without recording the chosen value. An implementer reading only §13.4 cannot tell what was decided | Owner correction; this plan works from §17, which does record the values |
| `OBS-02` | `D-22`/`OQ-015` combine two decisions (hashing algorithm and Cloudflare Access). §17 records "app login only" but `D-22`'s cell lists three models | Split the row, or annotate it |
| `OBS-03` | `D-09` reads "None selected — evaluation only" with status Confirmed, while `OQ-020` records "custom admin shell". These are consistent but read as a contradiction | Reword `D-09` to "Custom shell, no third-party template" |
| `OBS-04` | `.wrangler/verify-*.ps1` are referenced by the brief but absent from the workspace | Re-attach, or delete the reference |
| `OBS-05` | `AGENTS.md` §9 correctly says no test tooling exists. This plan must not be read as claiming otherwise | `OD-13` introduces tooling in Phase 5 |
| `OBS-06` | PRD §18.1 P1 exit includes `AC-073` (preview isolation), which cannot be met without two provisioned databases | Declared `Deferred` → P6 in §7.1; the PRD exit list is optimistic here |

---

## 9. Requirement traceability (compact)

| Requirement block | Phase | Notes |
|---|---|---|
| `FR-001`–`FR-019` (shell) | P1, P2 | `FR-016` in P1; the rest in P2A |
| `FR-020`–`FR-039` (content pages) | P2 | Data-dependent parts close at P4 exit |
| `FR-040`–`FR-059` (products) | P2B, P4 | Public rendering P2B; validation/editor P4.7 |
| `FR-060`–`FR-078` (blog) | P2B, P4 | `FR-074` (RSS) is COULD → P6 backlog if time allows |
| `FR-080`–`FR-099` (admin/auth) | P4 | `FR-097` verified in P4.16 and again in P5 |
| `FR-100`–`FR-118` (media) | P4 | `FR-106`/`FR-115` need `OD-07` |
| `FR-120`–`FR-139` (settings) | P4 | `FR-125` needs `OD-08` |
| `FR-140`–`FR-159` (SEO) | P3, P5 | `FR-150` needs P4.11 |
| `FR-160`–`FR-179` (publishing) | P4 | `FR-176`/`FR-177` are SHOULD |
| `FR-180`–`FR-199` (platform) | P1, P4, P6 | `FR-197`/`FR-191` scheduled jobs in P6 |
| `NFR-001`–`NFR-021` | P5 | `NFR-004` also checked in P2.12 |
| `NFR-041`–`NFR-060` | P1, P4, P5, P6 | `NFR-042` benchmark at P4.4 |

---

## 10. What this plan does not authorise

- It does not scaffold, install, migrate, deploy, commit or push.
- It does not create or modify any Cloudflare resource, DNS record, zone, database, bucket, secret or token.
- It does not modify `PRD.md`, `AGENTS.md`, or `.gitignore`.
- It does not close any gate. Phase 0 stays at **0 of 9** until the owner supplies evidence.
- It does not select the rich text editor, the island framework, the test toolchain, or the purge mechanism — those are `OD-05`, `OD-06`, `OD-13`, `OD-08`.

**Next action:** owner answers `OD-01`–`OD-04` and `OD-11`. Nothing else can start before that.
