# PRD — Garda Jaya CCTV Website & Admin CMS

**Product:** Garda Jaya CCTV — business website + admin CMS
**Document version:** v0.2
**Date:** 2026-10-26
**Status:** **Draft v0.2 — awaiting owner approval**
**Owner:** Garda Jaya CCTV (business owner)
**Prepared by:** Documentation phase — no implementation performed
**Supersedes:** v0.1 (Vercel + Next.js + Turso/libSQL target). See §1.4 and §13.7.

---

## 1. Document purpose, status, audience, change log, ID conventions

### 1.1 Purpose

This document is the single source of truth for *what* will be built, *why*, and *under which constraints*. It exists so any agent or developer can implement the product without re-deriving decisions from chat history, and so the business owner can approve or reject each unresolved item explicitly.

This PRD is **documentation only**. No application code, dependency, schema, or configuration exists yet. Nothing here authorises installing packages, creating a database, or selecting a vendor.

### 1.2 Status

Draft v0.2 — awaiting owner approval. Every item marked **Proposed** or **Open** (see the §13.4 decision table and §17) requires the owner's explicit sign-off before implementation. Items marked **Confirmed** reflect decisions already given by the owner.

**This document has never been approved.** v0.2 records a platform pivot mandated by the owner (§1.4). No requirement in it is authorised for implementation until the Phase 0 gate in §18.1 passes.

### 1.3 Audience

| Audience | What they need from this document |
|---|---|
| Business owner (non-technical) | Scope, what they will be able to edit, what is explicitly out of scope, vendor decisions needing approval |
| Implementing agent / developer | Requirements with stable IDs, constraints, acceptance criteria, phase plan |
| Designer / brand contributor | Brand rules, logo asset gaps, accessibility and responsive expectations |
| Future maintainer | Why the stack was chosen, what was rejected and why, what remains undecided |

### 1.4 Change log

| Version | Date | Author | Change |
|---|---|---|---|
| v0.1 | 2026-10-26 | Documentation phase | Initial draft: scope, requirement registers, persistence decisions, risks, open questions, phased delivery. Targeted Vercel + Next.js + Turso/libSQL. **Superseded by v0.2.** |
| v0.2 | 2026-10-26 | Documentation phase | **Hosting switched from Vercel to Cloudflare Workers; DB from Turso/libSQL to D1; media from provider-agnostic to R2; framework re-evaluated.** Added §12.1–§12.9 (Workers runtime model + framework evaluation matrix), rewrote §13 for D1/R2 with a Confirmed/Proposed/Open decision log and a superseded-decisions register (§13.7), added `FR-180`–`FR-199`, `NFR-041`–`NFR-060`, `AC-061`–`AC-080`, `DEP-011`–`DEP-016`, `RISK-011`–`RISK-018`, `OQ-013`–`OQ-020`, `D-17`–`D-26`. No requirement was deleted or renumbered. |
| v0.2.1 | 2026-10-26 | Documentation phase | **v0.2.1 — factual-accuracy corrections; no requirement scope change.** Re-verified the framework citations against the live Cloudflare documentation and the npm registry; replaced ellipsis citations with full URLs; re-sourced the `nodejs_compat` default claim to the compatibility-flags reference; removed bare re-assertions of volatile platform limits in favour of a single labelled source (§12.2); corrected FR cross-references in §12.9/§13, the `RISK-014` reference in §11.1, ISR vocabulary in §8.2 and `AC-043`/`AC-047`, the `NFR-019` change-record row, and the `.gitignore` contradiction in §2.6. No requirement was added, deleted, renumbered, or re-scoped. |

**Superseded decisions (v0.1 → v0.2), at a glance.** Full detail and rationale in §13.7.

| v0.1 decision | Reason for supersession | Replaced by |
|---|---|---|
| Vercel hosting | Owner decision: runtime is Cloudflare Workers | Workers (§12.1) |
| Turso (libSQL) as production DB | Owner decision: D1 is the database | Cloudflare D1 (§13.2) |
| Vercel Blob / Cloudinary as media | Owner decision: R2 is the media store | Cloudflare R2 (§13.3) |
| ISR (Incremental Static Regeneration) | Vercel-specific; no equivalent on Workers | On-demand SSR + HTTP cache + explicit purge (§12.4) |
| Next.js App Router as framework | Re-evaluated against the Workers target | One of three candidates in §12.7 — **not** selected (§12.9) |

### 1.5 How to read requirement IDs

Every requirement has a stable, never-reused ID. IDs are referenced by acceptance criteria (§15) and by agents when planning work.

| Prefix | Meaning | Range reserved | Highest used in v0.2 |
|---|---|---|---|
| `FR-###` | Functional requirement | `FR-001`–`FR-199` | `FR-199` |
| `NFR-###` | Non-functional requirement | `NFR-001`–`NFR-060` | `NFR-060` |
| `AC-###` | Acceptance criterion | `AC-001`–`AC-080` | `AC-080` |
| `DEP-###` | Dependency | `DEP-001`–`DEP-016` | `DEP-016` |
| `RISK-###` | Risk | `RISK-001`–`RISK-018` | `RISK-018` |
| `OQ-###` | Open question requiring owner decision | `OQ-001`–`OQ-020` | `OQ-020` |
| `D-##` | Decision-log entry (status: Confirmed / Proposed / Open) | `D-01`–`D-26` | `D-26` |

`D-##` entries live in the decision table in §13.4; `OQ-###` entries live in §17. An `OQ` asks the owner a question; a `D` records the resulting state. An OQ may be **Resolved** while its linked D stays **Confirmed**; a superseded D is marked in §13.7 and never deleted.

**Requirement ID index (reserved blocks — do not renumber, do not reuse retired IDs):**

| Block | Domain |
|---|---|
| `FR-001`–`FR-019` | Public site shell, navigation, language, global behaviour |
| `FR-020`–`FR-039` | Homepage, Tentang Kami, Layanan, Kontak, Kebijakan Privasi |
| `FR-040`–`FR-059` | Products (catalog, detail, pricing display, inquiry) |
| `FR-060`–`FR-079` | Blog / articles (last used: `FR-078`; `FR-079` reserved) |
| `FR-080`–`FR-099` | Admin dashboard & authentication |
| `FR-100`–`FR-119` | Media & uploads (last used: `FR-118`; `FR-119` reserved) |
| `FR-120`–`FR-139` | Settings, branding, site configuration |
| `FR-140`–`FR-159` | SEO |
| `FR-160`–`FR-179` | Publishing workflow, status model, redirects (last used: `FR-179`; `FR-171`–`FR-175` never issued and stay reserved) |
| `FR-180`–`FR-199` | **v0.2** — Cloudflare platform: Workers runtime, D1 migrations, R2 media pipeline, caching, deployment, observability |

Rules:

1. A requirement is never deleted. If dropped, its status becomes *retired* and the ID is never reused. Unused numbers inside a block remain reserved and must not be reassigned to another domain. (In this register `FR-135`, `FR-138` and `FR-171`–`FR-175` were never issued; they remain reserved and are **not** reused by the v0.2 block, which starts at `FR-180`.) The same applies to the non-functional register: **`NFR-022`–`NFR-040` were never issued.** They remain reserved and must not be reassigned; the v0.2 NFR block deliberately starts at `NFR-041` so that the gap stays visible and auditable rather than being silently backfilled.
2. Priority: **MUST** (launch blocker), **SHOULD** (expected at launch, may slip with owner approval), **COULD** (may move to the later-phase backlog).
3. Phase values refer to §18 (`P0`–`P6`).
4. Every `FR-###` is verified by at least one `AC-###`; §15.1 is the coverage map and must be updated whenever an FR is added.

---

## 2. Business context & objectives

### 2.1 Business context

Garda Jaya CCTV is an Indonesian business selling, installing, and servicing CCTV systems for **residential and commercial** customers. It currently has no website. Its brand identity exists only as raster logo files (§10).

The owner is not a developer. The commercial problem is not "we need a website" but "we need a website the owner can keep current without paying a developer for every text, photo, product, or price change." Products, articles, branding, and SEO settings must therefore be **data**, not code.

### 2.2 Primary objectives

| # | Objective | Success signal measurable at launch |
|---|---|---|
| O1 | Present a credible, professional business presence | All public pages published, no placeholder text, consistent brand usage |
| O2 | Let the owner publish and edit products, articles, branding, and SEO settings with **zero code edits** | Every content type editable from `/admin`; no deploy required to change copy or price |
| O3 | Generate inquiries through WhatsApp and a contact form | WhatsApp CTA on every page; prefilled message carries page/product context |
| O4 | Be discoverable for relevant local search intent | Clean indexable HTML, sitemap, structured data, per-page metadata — *no ranking guarantees* |
| O5 | Stay maintainable and low-cost for a small business | Cloudflare Workers + D1 + R2 on documented plan tiers, with adapter boundaries and documented swap points |

### 2.3 Secondary objectives

| # | Objective |
|---|---|
| S1 | Provide a reusable content foundation (categories, media library, SEO defaults) so growth does not require rework |
| S2 | Keep the system simple enough for one administrator to operate |
| S3 | Leave an honest audit trail of content changes |

### 2.4 Brand direction

| Token | Value | Usage |
|---|---|---|
| Deep Navy | `#102A43` | Primary dark surface, headings, footer, logo navy |
| Security Blue | `#1D4ED8` | Primary action, links, focus ring |
| Sky Blue | `#38BDF8` | Accent, highlights, illustration tint |
| White | `#FFFFFF` | Page surface, reversed logo plate |
| Neutrals | To be derived | Body text, borders, muted surfaces, table stripes |

Direction: calm, technical, trustworthy. Sky Blue is an accent only. The public site must read as a *business* site, never as a software dashboard (§7.9).

Colour values are brand tokens; contrast ratios must be verified against WCAG 2.1 AA (`NFR-003`) before shipping. Sky Blue on white is unlikely to pass for text and must not carry body copy or essential small text.

### 2.5 Target audience

| Segment | Description | Primary need |
|---|---|---|
| Residential prospect | Homeowner in Indonesia researching home security | Understand offerings, trust the business, ask for a quote on WhatsApp |
| Commercial prospect | Shop/office/warehouse decision maker | Product specs, service capability, credible contact channel |
| Existing customer | Already installed, needs service | Fast path to contact/service information |
| Business owner (admin) | Non-technical, single operator | Edit content, publish articles, upload photos, change the WhatsApp number |

### 2.6 Current repository state (re-verified 2026-10-26, during the v0.2 revision)

The repository was re-audited with file tools and Git before this revision was written. This section replaces the v0.1 text, which is now inaccurate in two respects (it claimed `AGENTS.md` was absent and that there were zero commits).

- **Present:** `AGENTS.md`, `PRD.md`, and `logo/` containing exactly 4 PNG files (§10.1). Nothing else.
- **Absent:** no `README.md`, no `package.json`, no lockfile, no `node_modules`, no `wrangler.jsonc`/`wrangler.toml`, no `migrations/`, no `.dev.vars`, no `.env*`, no `tsconfig.json`, no framework config, no `src/`, no `app/`, no `public/`, no scripts, no tests, no CI configuration, no Cloudflare resource of any kind. (`.gitignore` is **not** absent — see the post-audit note below.)
- **Git (verified, not assumed):** branch `main` exists and has **exactly one commit** — `342f0c1 "Add new logo files for Garda Jaya CCTV branding"`. `git ls-files` tracks `AGENTS.md`, `PRD.md`, and three of the four logo PNGs. At audit time `git status --porcelain` reported a single untracked path: `logo/Garda Jaya CCTV Logo Presentation.png`, and there was **no `.gitignore`**.
- **Post-audit note:** an untracked `.gitignore` (one line, ignoring the presentation sheet) appeared in the working tree *after* the audit, from outside this documentation task. It is recorded here so the state described above is not mistaken for a contradiction. It is not authored by the documentation phase and does not constitute Phase 1 tooling work; a proper `.gitignore` covering `.dev.vars`, `.wrangler/`, build output and `node_modules` remains a Phase 1 deliverable.

Consequences binding all later work:

1. There are **no existing conventions to follow** and **no conflicting application files**. No application file was overwritten.
2. No build, lint, test, or dev command exists. Any command reference in planning documents is *planned*, not current. Do not present `wrangler dev`, `npm run build`, or any migration command as runnable today.
3. The documentation phase is **not** the initialisation phase: `.gitignore` (including `.dev.vars` and `.wrangler/`), the Cloudflare account/zone setup, D1 and R2 provisioning, and the project scaffold are all Phase 1 work (§18.1) and require the Phase 0 gate to pass first.
4. `logo/` is treated as an immutable source reference (§10.5).

---

## 3. Target users & use cases

### 3.1 Visitor personas

| Persona | Context | Goals | Frustrations to avoid |
|---|---|---|---|
| **Rumah Tangga** — "Bu Sari" | Mobile, evening, comparing 2–3 vendors | Know whether the business handles homes, see relevant products, ask price without a form marathon | Jargon, hidden pricing with no way to ask, slow mobile pages, dead WhatsApp links |
| **Usaha** — "Pak Andi" | Desktop, office hours, needs specs | Verify product capability and installation service, get a channel to ask | Missing specs, unclear service scope, no legitimacy signals |
| **Pelanggan Lama** — "Pak Yusuf" | Mobile, urgent (camera offline) | Reach a human quickly | Being funnelled through sales pages to find a phone number |

### 3.2 Admin persona

**"Admin Garda Jaya"** — the owner or a trusted staff member. Non-technical, uses a laptop and occasionally a phone. Needs to add a product after a new shipment, publish a promo article, replace the hero photo, change the WhatsApp number, adjust a page title for search. Expects Indonesian labels, immediate feedback, and no way to accidentally destroy published content.

### 3.3 Jobs to be done

| JTBD | Job | Served by |
|---|---|---|
| JTBD-1 | "When I need CCTV for my home, help me judge if this business is credible and can install it properly." | Homepage, Tentang Kami, Layanan (`FR-020`–`FR-027`) |
| JTBD-2 | "When I compare products, show me what it does and how to ask the price." | Catalog/detail, inquiry CTA (`FR-040`–`FR-049`) |
| JTBD-3 | "When I have a problem, get me to a human in one tap." | Global WhatsApp CTA, Kontak (`FR-007`, `FR-028`) |
| JTBD-4 | "When I want to learn before buying, give me useful articles." | Blog (`FR-060`–`FR-070`) |
| JTBD-5 | "When I add a product, publish it without calling a developer." | Admin products + publishing (`FR-040`–`FR-053`, `FR-160`–`FR-170`) |
| JTBD-6 | "When I change the phone number, it must change everywhere at once." | Settings (`FR-121`, `FR-125`) |

### 3.4 User journeys

**J1 — Residential inquiry (primary conversion path)**
Lands from search/social → homepage hero explains what the business does for homes and businesses → service highlights → taps "Konsultasi via WhatsApp" → WhatsApp opens in a new tab with a prefilled Indonesian message → conversation happens **in WhatsApp, not on the site**.

**J2 — Product-led inquiry**
Search result for a product type → `/produk` → filter by category → product detail with specs and price display (`fixed` / `from` / `range` / `inquiry`) → "Tanya Harga via WhatsApp" → prefilled message includes the product name and URL.

**J3 — Service discovery**
Searches for installation/service → `/layanan` → service detail describing scope → contact CTA.

**J4 — Content-led trust**
Blog article from search → body with correct heading hierarchy, images, author, date → related articles → CTA band → contact.

**J5 — Owner publishes a product**
`/admin/masuk` → dashboard → Produk → Tambah Produk → name, category, specs, optional price, images, SEO title → Simpan sebagai Draf → Pratinjau → Terbitkan → appears on `/produk` without a deploy.

**J6 — Owner renames a published article slug**
Edits slug → system warns the old URL will permanently redirect → saves → old URL returns HTTP 301 (`FR-150`, `FR-165`, `FR-178`).

---

## 4. Scope

### 4.1 In scope

| Area | Included |
|---|---|
| Public website | Beranda, Tentang Kami, Layanan (list + detail), Produk (catalog + filters + detail), Artikel (list + detail + category), Kontak, Kebijakan Privasi, 404 |
| Admin CMS | Authentication, dashboard, product management, article management, media library, site settings/branding, SEO settings, publishing workflow |
| Content model | Products, categories, articles, media, static page content, site settings, slug history |
| SEO | Metadata, canonical, OG/Twitter, sitemap, robots, JSON-LD, redirects, draft noindex |
| Persistence | SQLite-compatible relational model; production persistence via **Cloudflare D1**; media via **Cloudflare R2** (§13) |
| Non-functional | Performance, accessibility, responsive behaviour, security baseline, backup targets |

### 4.2 Explicit non-goals

Out of scope. Must not be built, scaffolded, or partially implemented without a new owner-approved PRD revision.

| # | Non-goal | Note |
|---|---|---|
| NG-1 | Checkout, cart, payments, order management | Inquiry-based business, not e-commerce |
| NG-2 | Customer accounts, visitor login, order history | Only the single Administrator logs in |
| NG-3 | Inventory / warehouse / stock levels | Availability is narrative text, not stock counts |
| NG-4 | Quotation engine, price calculators, PDF quotes | Quotations happen off-platform |
| NG-5 | Multi-language / language switcher | Indonesian only at launch; i18n-*readiness* only (`NFR-012`) |
| NG-6 | Mobile apps (iOS/Android) | Responsive web only |
| NG-7 | Complex RBAC, teams, per-field permissions | Single `Administrator` role (`FR-091`) |
| NG-8 | Marketplace sync (Tokopedia/Shopee), product feeds | Not integrated |
| NG-9 | AI content generation or AI chat assistant | Not included |
| NG-10 | Analytics dashboards, charts, visitor tracking UI | Dashboard shows content counts only (`FR-087`) |
| NG-11 | Live chat widget, ticketing, CRM | WhatsApp + contact form only |
| NG-12 | Customer reviews / testimonials system | Content would be unverifiable; see §6.7 |

---

## 5. Sitemap, journeys, navigation model

### 5.1 Public sitemap (Indonesian slugs)

| Path | Page | Indexable | Phase |
|---|---|---|---|
| `/` | Beranda | Yes | P4 |
| `/tentang-kami` | Tentang Kami | Yes | P4 |
| `/layanan` | Layanan (list) | Yes | P4 |
| `/layanan/[slug]` | Layanan detail | Published only | P4 |
| `/produk` | Produk (catalog) | Yes | P4 |
| `/produk/[slug]` | Produk detail | Published only | P4 |
| `/produk/kategori/[slug]` | Produk per kategori | Yes | P4 |
| `/blog` | Artikel (list) | Yes | P4 |
| `/blog/[slug]` | Artikel detail | Published only | P4 |
| `/blog/kategori/[slug]` | Artikel per kategori | Yes | P4 |
| `/kontak` | Kontak | Yes | P4 |
| `/kebijakan-privasi` | Kebijakan Privasi | Yes | P4 |
| `/sitemap.xml` | Generated sitemap | n/a | P5 |
| `/robots.txt` | Generated robots | n/a | P5 |
| unmatched | 404 (Bahasa Indonesia) | n/a | P4 |

### 5.2 Admin sitemap (Indonesian labels)

| Path | Screen |
|---|---|
| `/admin/masuk` | Masuk |
| `/admin` | Dasbor |
| `/admin/produk` | Produk — daftar |
| `/admin/produk/baru` | Produk — tambah |
| `/admin/produk/[id]` | Produk — ubah |
| `/admin/produk/kategori` | Kategori produk |
| `/admin/blog` | Artikel — daftar |
| `/admin/blog/baru` | Artikel — tambah |
| `/admin/blog/[id]` | Artikel — ubah |
| `/admin/blog/kategori` | Kategori artikel |
| `/admin/media` | Media |
| `/admin/pengaturan` | Pengaturan — identitas situs |
| `/admin/pengaturan/kontak` | Pengaturan — kontak & WhatsApp |
| `/admin/pengaturan/branding` | Pengaturan — branding & logo |
| `/admin/pengaturan/seo` | Pengaturan — SEO default |
| `/admin/keluar` | Keluar |

The `/admin` prefix is fixed. All admin routes are excluded from indexing and from `sitemap.xml`.

### 5.3 Navigation model

Primary header navigation (max 6 items): `Beranda` · `Tentang Kami` · `Layanan` · `Produk` · `Artikel` · `Kontak`, plus a persistent WhatsApp action.

- Mobile: hamburger drawer, focus-trapped, closable with `Esc`, `aria-expanded` maintained (`FR-009`).
- Footer: identity, short description, contact block, quick links, privacy link, copyright year computed at render.
- Navigation resolves from published data only; a category with zero published items is not linked (`FR-006`).
- No navigation item may point to a page that does not exist.

### 5.4 Functional requirements — site shell & global behaviour (`FR-001`–`FR-019`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-001` | Global header with primary navigation and a persistent WhatsApp action, rendered on every public page | MUST | P4 | `AC-001` |
| `FR-002` | Global footer with site identity, contact block (phone, email, address), quick links, privacy link, computed copyright year | MUST | P4 | `AC-001` |
| `FR-003` | All public copy in Bahasa Indonesia; document `lang="id"` | MUST | P4 | `AC-002` |
| `FR-004` | No language switcher and no second locale in the public UI | MUST | P4 | `AC-002` |
| `FR-005` | Custom 404 page in Bahasa Indonesia with navigation home and WhatsApp CTA | MUST | P4 | `AC-003` |
| `FR-006` | Navigation and listings are generated from published data only; empty categories are not linked | MUST | P4 | `AC-004` |
| `FR-007` | WhatsApp CTA built from the CMS-configured number: `https://wa.me/<number>?text=<urlencoded>`, new tab, `rel="noopener noreferrer"`, copy never implies automatic sending | MUST | P4 | `AC-005` |
| `FR-008` | Breadcrumbs on all inner pages, each link resolving to a real page | SHOULD | P4 | `AC-004` |
| `FR-009` | Mobile navigation drawer: focus trap, `Esc` to close, `aria-expanded` state, scroll lock | MUST | P4 | `AC-001` |
| `FR-010` | Skip link as first focusable element; `header`/`nav`/`main`/`footer` landmarks; one `h1`, no skipped heading levels | MUST | P5 | `AC-006` |
| `FR-011` | Current navigation item marked with `aria-current="page"` and a visible style | SHOULD | P4 | `AC-001` |
| `FR-012` | URLs normalised to lowercase; mixed-case requests redirect permanently to the canonical lowercase URL | COULD | P5 | `AC-004` |
| `FR-013` | Production build contains no lorem ipsum, `TODO`, "Coming soon", or placeholder business claims | MUST | P4 | `AC-003` |
| `FR-014` | Optional sticky WhatsApp action on mobile that does not obscure content or focus targets | COULD | P4 | `AC-007` |
| `FR-015` | Primary page content is present in server-rendered HTML; core reading and navigation do not require JavaScript | MUST | P4 | `AC-002` |
| `FR-016` | Locale (`id-ID`) and timezone (`Asia/Jakarta`) defined once as central constants and used for all formatting | MUST | P1 | `AC-007` |
| `FR-017` | Brand tokens (navy/blue/sky/white/neutrals) applied consistently across public pages | MUST | P4 | `AC-007` |
| `FR-018` | On client-side route change, focus resets to the main region and scroll position is handled | COULD | P4 | `AC-006` |
| `FR-019` | All external links open safely with `rel="noopener noreferrer"` where `target="_blank"` | SHOULD | P4 | `AC-001` |

---

## 6. Public website requirements

Global rules for every public page: Bahasa Indonesia copy, `lang="id"`, server-rendered content, responsive from 320 px, semantic landmarks, visible focus states, and no admin UI elements.

### 6.1 Homepage (`/`)

Homepage is composed of CMS-editable blocks rendered in a fixed order. Blocks can be hidden by the admin but not reordered arbitrarily in MVP.

| Block | Content | Requirement |
|---|---|---|
| B1 Hero | Headline, subheadline, primary CTA (WhatsApp), secondary CTA (Lihat Produk), optional image | `FR-020` |
| B2 Trust strip | 3–4 short factual statements supplied by the owner — **no certifications, counts, years, or warranties** | `FR-024` |
| B3 Layanan unggulan | 3–6 service cards linking to `/layanan/[slug]` | `FR-021` |
| B4 Produk unggulan | Admin-selected published products | `FR-022` |
| B5 Artikel terbaru | Latest 3 published articles | `FR-023` |
| B6 CTA band | Headline + WhatsApp button + phone | `FR-025` |

A block with no eligible content is omitted entirely — never rendered as an empty shell or with placeholder text (`FR-039`).

### 6.2 Tentang Kami (`/tentang-kami`)

Sections: who the business is, what it does (sales/installation/service), coverage statement, and how to start a conversation. Body is CMS-editable (`FR-026`, `FR-031`). Any factual claim (founding year, project counts, certifications, partnerships) must come from the owner; the system must not prefill such claims (`FR-038`).

### 6.3 Layanan (`/layanan`, `/layanan/[slug]`)

Service list with cards. Each detail page contains description, what is included, process steps, and a contact CTA. Services are CMS records with slug, title, summary, body, process steps, optional image, and status (`FR-027`, `FR-037`).

### 6.4 Produk (`/produk`, `/produk/[slug]`, `/produk/kategori/[slug]`)

- Catalog grid: image, name, brand, category, price display per §6.4.1, short summary.
- Filters: category (required), brand (should), search (could), sort (could). Filters update URL query so results are shareable and back/forward works.
- Pagination: server-side, 12 per page default, configurable, stable ordering.
- Empty state with a helpful message and WhatsApp CTA — never a blank grid.
- Detail page: gallery, name, brand, category, price display, specs table, description, related products, inquiry CTA.
- Draft and archived products are never publicly reachable — not in lists, not by direct URL (404, not a preview).

#### 6.4.1 Price display rules (critical)

Pricing is **inquiry-based** by default. `price` is optional. The renderer branches on `price_display`:

| `price_display` | Required data | Rendering (Bahasa Indonesia) |
|---|---|---|
| `fixed` | `price_amount` | `Rp 1.250.000` |
| `from` | `price_amount` | `Mulai dari Rp 1.250.000` |
| `range` | `price_amount`, `price_max` | `Rp 1.250.000 – Rp 2.000.000` |
| `inquiry` | none | `Hubungi kami untuk harga` (no number rendered) |

Hard rules: never render an empty price, never render `Rp 0`, never render a range where `max < min`, never invent a number to fill a layout gap. If `price_display` is `fixed`/`from`/`range` but the amount is missing, the record is invalid and cannot be published (`FR-162`); the public renderer falls back to `inquiry` rather than showing a broken price (`FR-046`).

### 6.5 Blog (`/blog`, `/blog/[slug]`, `/blog/kategori/[slug]`)

- List: cover image, title, excerpt, category, publish date, pagination.
- Detail: title, publish date, author display name, body with `h2`/`h3` hierarchy, images with alt text, optional table of contents, related articles.
- Draft articles return 404 publicly and are `noindex` in preview (`FR-065`, `FR-149`).

### 6.6 Kontak (`/kontak`)

- Contact block: address (as provided), phone, email, operating hours, map embed **only if** a valid URL is configured.
- WhatsApp CTA using the CMS number (`FR-007`).
- Contact form: name, contact (phone or email), message, optional product reference. Server-validated, rate-limited, spam-protected, stores minimal fields.
- Success message states a reply is attempted during operating hours — **must not promise an automatic response or a specific response time** (`FR-034`).
- No public exposure of internal configuration, stack traces, or admin internals.

### 6.7 Trust-building without fabricated claims

Trust must come from clarity, not invented social proof. Permitted: clear description of services and process, transparent contact channels, real product specifications and photos supplied by the owner, privacy policy.

Prohibited unless the owner supplies verifiable evidence and approves in writing: certification badges, awards, client logos, named testimonials, review counts, star ratings, "X tahun pengalaman", "X proyek selesai", "garansi resmi X tahun", partner/authorised-dealer claims, and stock-photo "our team" imagery presented as real staff (`FR-024`, `FR-152`, `RISK-006`).

### 6.8 Kebijakan Privasi (`/kebijakan-privasi`)

CMS-editable page describing what data is collected (contact form fields, server logs), why, retention, and how to request deletion. May reference Indonesian personal-data protection law (UU PDP No. 27/2022) as context — **not** as a compliance certification. Legal wording requires owner/counsel review (`OQ-008`).

### 6.9 Responsive behaviour

| Breakpoint | Width | Layout expectation |
|---|---|---|
| Mobile S | 360 px | Single column, drawer nav, 44×44 px touch targets, sticky WhatsApp allowed |
| Mobile L | 430 px | Single column, larger type scale |
| Tablet | 768 px | 2-column grids, persistent nav optional |
| Laptop | 1024 px | 3-column product grid, sidebar filters |
| Desktop | 1280 px | Max content width ~1200 px, 4-column product grid |
| Wide | 1536 px+ | Centred container, no content stretch |

No horizontal scrolling at any width ≥ 320 px. Images use `srcset`/`sizes` and never overflow their container.

### 6.10 Functional requirements — content pages (`FR-020`–`FR-039`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-020` | Homepage hero (headline, subheadline, two CTAs, optional image) editable in the CMS | MUST | P4 | `AC-008` |
| `FR-021` | Homepage service highlights block driven by published services | MUST | P4 | `AC-008` |
| `FR-022` | Homepage featured-products block driven by the product featured flag, published only | SHOULD | P4 | `AC-008` |
| `FR-023` | Homepage latest-articles block showing the 3 most recent published articles | SHOULD | P4 | `AC-008` |
| `FR-024` | Homepage trust strip accepts only owner-supplied factual statements; prohibited claim types are blocked in editor guidance and reviewed before launch | MUST | P4 | `AC-009` |
| `FR-025` | Homepage CTA band with WhatsApp and phone actions from settings | MUST | P4 | `AC-009` |
| `FR-026` | Tentang Kami page with CMS-editable body, rendered server-side | MUST | P4 | `AC-010` |
| `FR-027` | Layanan list and detail pages generated from published service records | MUST | P4 | `AC-011` |
| `FR-028` | Kontak page with address, phone (`tel:`), email (`mailto:`), hours, WhatsApp CTA | MUST | P4 | `AC-012` |
| `FR-029` | Contact form with server-side validation, rate limiting, spam protection, and storage of minimal fields | SHOULD | P4 | `AC-013` |
| `FR-030` | Kebijakan Privasi page, CMS-editable, linked in footer, present in sitemap | MUST | P4 | `AC-014` |
| `FR-031` | Static page content (Tentang Kami, Kebijakan Privasi, block copy) editable without a deploy | MUST | P4 | `AC-010` |
| `FR-032` | Operating hours displayed and formatted in `Asia/Jakarta` | SHOULD | P4 | `AC-012` |
| `FR-033` | Service-area statement rendered only from owner-supplied text; no invented coverage | MUST | P4 | `AC-010` |
| `FR-034` | Contact-form and CTA copy must not promise an automatic reply or a specific response time | MUST | P4 | `AC-013` |
| `FR-035` | Contact submissions store only necessary fields; IPs used for throttling are hashed; no full PII in logs | MUST | P4 | `AC-013` |
| `FR-036` | Map embed rendered only when a valid maps URL is configured; otherwise omitted cleanly | SHOULD | P4 | `AC-012` |
| `FR-037` | Service detail shows ordered process steps and a "what is included" list | SHOULD | P4 | `AC-011` |
| `FR-038` | Editors must not prefill unverifiable business facts (founding year, counts, certifications) in any template or seed | MUST | P4 | `AC-009` |
| `FR-039` | Empty homepage blocks are omitted entirely (no empty container, no placeholder) | MUST | P4 | `AC-008` |

### 6.11 Functional requirements — products (`FR-040`–`FR-059`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-040` | Product catalog grid with server-side pagination (default 12 per page, configurable) | MUST | P4 | `AC-015` |
| `FR-041` | Category filter persisted in the URL query; back/forward restores results | MUST | P4 | `AC-016` |
| `FR-042` | Brand filter persisted in the URL query, combinable with category | SHOULD | P4 | `AC-016` |
| `FR-043` | Keyword search over product name/summary, URL-persisted, combinable with filters | COULD | P4 | `AC-016` |
| `FR-044` | Empty catalog/filter state shows a helpful message and WhatsApp CTA | MUST | P4 | `AC-015` |
| `FR-045` | Product detail gallery with cover first and all images in authored order | MUST | P4 | `AC-017` |
| `FR-046` | Price rendering branches on `price_display`; never renders an empty price or `Rp 0`; falls back to inquiry text when data is incomplete | MUST | P4 | `AC-018` |
| `FR-047` | "Tanya Harga via WhatsApp" CTA with prefilled message containing product name and canonical URL | MUST | P4 | `AC-019` |
| `FR-048` | Specifications rendered as an ordered label/value table | MUST | P4 | `AC-017` |
| `FR-049` | Product description present in server-rendered HTML | MUST | P4 | `AC-017` |
| `FR-050` | Related products from the same category, excluding the current product, published only | SHOULD | P4 | `AC-020` |
| `FR-051` | Draft, unpublished, and archived products return 404 publicly and appear in no listing or sitemap | MUST | P4 | `AC-020` |
| `FR-052` | Catalog ordering is deterministic and stable across reloads | SHOULD | P4 | `AC-015` |
| `FR-053` | Per-product SEO title, description, canonical, and OG tags rendered from CMS fields | MUST | P5 | `AC-020` |
| `FR-054` | Category archive pages list only published products in that category | SHOULD | P4 | `AC-020` |
| `FR-055` | Product JSON-LD emitted; `offers` included only when a real price exists | MUST | P5 | `AC-020` |
| `FR-056` | Product images require non-empty alt text before the product can be published | MUST | P4 | `AC-020` |
| `FR-057` | Optional price note (e.g. "belum termasuk instalasi") rendered adjacent to the price display | SHOULD | P4 | `AC-018` |
| `FR-058` | Featured flag drives homepage placement and can be toggled without a deploy | SHOULD | P4 | `AC-020` |
| `FR-059` | Sort options (newest, name A–Z) persisted in the URL | COULD | P4 | `AC-016` |

### 6.12 Functional requirements — blog (`FR-060`–`FR-078`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-060` | Article list with pagination and stable ordering by publish date | MUST | P4 | `AC-021` |
| `FR-061` | Article category filter and archive routes | MUST | P4 | `AC-021` |
| `FR-062` | Article body rendered in server-side HTML | MUST | P4 | `AC-022` |
| `FR-063` | Author display name and publish date shown, date formatted in `Asia/Jakarta` | MUST | P4 | `AC-022` |
| `FR-064` | Related articles exclude the current article and are published only | SHOULD | P4 | `AC-023` |
| `FR-065` | Draft/unpublished articles return 404 publicly and appear in no sitemap or listing | MUST | P4 | `AC-024` |
| `FR-066` | Empty blog state with a helpful message (no blank page) | SHOULD | P4 | `AC-021` |
| `FR-067` | Table of contents for articles above a configured length, linking to real heading anchors | COULD | P4 | `AC-022` |
| `FR-068` | Body images require non-empty alt text before publishing | MUST | P4 | `AC-023` |
| `FR-069` | Article category archive lists only published articles | MUST | P4 | `AC-021` |
| `FR-070` | Editor output enforces heading hierarchy (`h2`/`h3` only; `h1` reserved for the title) | MUST | P4 | `AC-022` |
| `FR-071` | Reading-time estimate displayed when body length exceeds the threshold | COULD | P4 | `AC-025` |
| `FR-072` | Share actions (WhatsApp, copy link) that work without third-party trackers | COULD | P4 | `AC-025` |
| `FR-073` | `BlogPosting` JSON-LD emitted for published articles | MUST | P5 | `AC-023` |
| `FR-074` | Optional RSS/Atom feed of published articles | COULD | P6 | `AC-025` |
| `FR-075` | Cover image used for OG; article without cover falls back to the default OG image | SHOULD | P5 | `AC-023` |
| `FR-076` | Excerpt auto-derived from body when left empty (and editable) | COULD | P3 | `AC-025` |
| `FR-077` | Previous/next article navigation within the same category | COULD | P4 | `AC-025` |
| `FR-078` | Blog keyword search over title/excerpt, URL-persisted | COULD | P4 | `AC-025` |

---

## 7. Admin CMS requirements

The admin panel is a separate visual world: dense, functional, Indonesian-labelled, and never linked from public navigation. `/admin/masuk` is the only entry point and is `noindex`.

### 7.1 Dashboard (`/admin`)

Purpose: orient the owner in five seconds and route them to work.

- Counts: total/published/draft products; total/published/draft articles.
- "Terakhir diperbarui": five most recently updated records with type, title, and `Asia/Jakarta` timestamp.
- Quick links: Tambah Produk, Tulis Artikel, Unggah Media, Pengaturan.
- No charts, no traffic analytics, no vanity metrics (`NG-10`).

### 7.2 Products (`/admin/produk`)

List with search, category filter, status filter, sort by updated date. Row actions: Ubah, Pratinjau, Terbitkan/Nonaktifkan, Arsipkan.

Editor fields: name, slug (auto-derived, editable), category (required), brand, short summary, description (rich text), specs (ordered key/value rows), `price_display`, `price_amount`, `price_max`, `price_note`, images (ordered, one cover), featured flag, SEO title/description, status.

Validation: name required, category required, slug unique and URL-safe, `range` requires `max ≥ min`, `fixed`/`from`/`range` require an amount, every image requires alt text. Save is blocked with field-level Indonesian messages.

### 7.3 Blog (`/admin/blog`)

List with status filter and search. Editor: title, slug, excerpt, cover image, category, body (rich text), author display name (from settings), publish date, SEO fields, status.

### 7.4 Media (`/admin/media`)

Grid library with search, type filter, upload, and pagination. Shows thumbnail, filename, dimensions, size, alt text, and usage count. Actions: edit alt text, replace file, delete with reference warning. See §7.7 and §13.3.

### 7.5 Settings (`/admin/pengaturan`)

Grouped forms with explicit save, success toast, and a visible "last saved" timestamp.

| Group | Fields |
|---|---|
| Identitas situs | Site name, tagline, short description, logo slot assignments |
| Kontak & WhatsApp | Phone, WhatsApp number, email, address, maps URL, operating hours, social links |
| Branding | Brand colour tokens (validated), logo variants, favicon/app-icon slots |
| SEO default | Default title template, default meta description, default OG image, verification token |

Changes must appear on the public site without a redeploy (`FR-125`).

### 7.6 SEO (`/admin/pengaturan/seo` + per-record fields)

Global defaults plus per-record overrides for products, articles, services, and static pages. Live preview of the rendered title/description with character guidance (guidance, not hard truncation).

### 7.7 Media handling rules

Uploads are validated, streamed to **Cloudflare R2**, and referenced by a public URL on an owner-controlled custom domain. No upload is ever written to the Worker's filesystem as the source of truth — the Workers filesystem is a per-request memory VFS whose `/tmp` does not survive the request (§12.2). Deleting referenced media warns and requires confirmation.

Server-side image *transformation* is **not** part of this phase: `sharp` and similar native image libraries are not viable under the Workers CPU budget. Validation (magic bytes, size, dimensions), EXIF stripping and pre-generated variants are in scope; on-the-fly resizing is an **Open** decision (§13.4 `D-19`, `OQ-014`).

### 7.8 Rich text editor requirements (library not committed)

Must support: WYSIWYG editing; headings `h2`/`h3` only; bold/italic; ordered/unordered lists; links with external-link `rel` handling; inline images from the media library; tables; blockquote; undo/redo; paste-from-Word sanity (strip `mso-*` styles and inline font soup); preview mode.

Output is either allowlist-sanitised HTML (re-sanitised server-side on save and on render) or a structured JSON document rendered to safe HTML. Storage format is **Open** (`OQ-005`). Candidate libraries for evaluation: Tiptap, Lexical, Markdown + preview. No selection is made here.

### 7.9 Admin shell

Responsive layout with collapsible sidebar, top bar with page title and logout, breadcrumb, and content area. Usable at 360 px for quick edits. Admin styling must not leak into public bundles — no shared admin CSS on public pages (`RISK-009`).

### 7.10 Admin template evaluation gate

> This is the "§7 gate" referenced in project briefs. **Criteria and evaluation only** — no template is selected, installed, or committed in this phase.

#### 7.10.1 Mandatory gate (all must pass)

| # | Criterion | How to verify |
|---|---|---|
| G1 | **Free use AND commercial use permitted** | Read the `LICENSE` file in the repository — not the README, not the marketing page. Public visibility ≠ commercial-use permission. Record the licence name and the exact clause permitting commercial use |
| G2 | Framework compatibility | Works with the chosen framework (§12.2); no jQuery-only or server-rendered-PHP-only template |
| G3 | Maintenance status | Commits/releases within the last 6 months; critical issues triaged; no abandonment notice |
| G4 | Responsive behaviour | Usable at 360 px and 1440 px; sidebar collapses sanely |
| G5 | Accessibility & usability | Keyboard navigable, labelled forms, focus states, sensible contrast; a non-technical user can complete core tasks unaided |
| G6 | Branding customisation | Colours/typography adjustable without forking core template files |
| G7 | Dependency & security posture | Minimal dependencies, no known critical advisories, no obfuscated bundles, no telemetry/phone-home |
| G8 | Suitability for a small CMS | Provides the shell (sidebar, tables, forms, modals, toasts) without imposing an unwanted data layer or build system |
| G9 | **Public site must not look like an admin dashboard** | Template styles must not leak into public pages; separate layout and CSS boundaries required |

#### 7.10.2 Additional considerations

Attribution requirements (some "free" licences demand visible credit); premium upsell traps (a free tier gating essential components is not free here); component-library coupling (a template built on a paid component set is not free); styling approach conflicts (Tailwind vs plain CSS) with the public design system; long-term upgrade path without constant merge conflicts.

#### 7.10.3 Examples to evaluate (no commitment — verify every licence)

| Candidate | What to verify |
|---|---|
| Tabler | MIT historically — confirm `LICENSE`; Bootstrap-based |
| AdminLTE | MIT historically — confirm `LICENSE`; older Bootstrap lineage |
| Flowbite Admin | Mixed licensing across components — verify carefully |
| shadcn/ui-based free dashboards | Component code is typically MIT — verify each dashboard repo's own `LICENSE` |
| Materio / Sneat free tiers | Free-tier scope and commercial-use terms must be read in the `LICENSE`/EULA |

**Warning:** licence terms differ between a UI kit, a dashboard repo, and a paid "pro" tier. A repository being public and popular does **not** establish commercial-use rights (`RISK-005`).

#### 7.10.4 Outcome of this phase

No template chosen. Decision remains **Open** (`D-09`, `OQ-006`). Evaluation completes in Phase 3 before the admin shell is built, with the licence finding recorded in this PRD.

### 7.11 Functional requirements — admin & auth (`FR-080`–`FR-099`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-080` | Login screen at `/admin/masuk`; authenticated area under `/admin` | MUST | P2 | `AC-026` |
| `FR-081` | Passwords stored as a slow, salted, one-way hash; never plaintext, never logged. **Algorithm Open** — Argon2id/bcrypt cost ≥ 12 may exceed the Workers CPU budget; PBKDF2-SHA256 via Web Crypto is the leading candidate pending benchmark (`OQ-015`; §13.8) | MUST | P2 | `AC-027` |
| `FR-082` | Every admin request re-validates the session server-side against the session store | MUST | P2 | `AC-028` |
| `FR-083` | Session cookie `HttpOnly`, `Secure` in production, `SameSite=Lax` or `Strict`, scoped path; no tokens in `localStorage` | MUST | P2 | `AC-029` |
| `FR-084` | CSRF protection on all state-changing requests (origin/`Sec-Fetch-Site` check plus per-session token) | MUST | P2 | `AC-030` |
| `FR-085` | Login throttling: 5 failed attempts per 15 minutes per IP+account, progressive delay, generic failure message | MUST | P2 | `AC-031` |
| `FR-086` | Absolute session expiry and server-side logout invalidation | MUST | P2 | `AC-032` |
| `FR-087` | Dashboard shows product and article counts split by status | MUST | P3 | `AC-033` |
| `FR-088` | Dashboard "Terakhir diperbarui" list of the 5 most recent updates | MUST | P3 | `AC-033` |
| `FR-089` | Dashboard quick links to core tasks | SHOULD | P3 | `AC-033` |
| `FR-090` | Admin layout usable at 360 px without horizontal scroll | MUST | P3 | `AC-033` |
| `FR-091` | Exactly one role, `administrator`; no RBAC, invitations, or permission matrix | MUST | P2 | `AC-034` |
| `FR-092` | Route middleware is a UX redirect only; each handler/API independently enforces authentication | MUST | P2 | `AC-028` |
| `FR-093` | Audit log of content and settings changes (actor, action, entity, UTC timestamp) | SHOULD | P3 | `AC-035` |
| `FR-094` | Admin account provisioned from environment configuration or a one-time setup step; no default credentials in the repo | MUST | P2 | `AC-034` |
| `FR-095` | Idle timeout (e.g. 2 hours) in addition to absolute expiry | SHOULD | P2 | `AC-032` |
| `FR-096` | Changing the password invalidates other active sessions | SHOULD | P2 | `AC-032` |
| `FR-097` | Admin routes are `noindex` and disallowed in `robots.txt` | MUST | P5 | `AC-036` |
| `FR-098` | Authentication errors are generic and do not reveal whether an account exists | MUST | P2 | `AC-026` |
| `FR-099` | All admin UI labels and messages in Bahasa Indonesia | SHOULD | P3 | `AC-036` |

### 7.12 Functional requirements — media & uploads (`FR-100`–`FR-118`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-100` | Upload through the media library with server-side validation before storage | MUST | P2 | `AC-037` |
| `FR-101` | Storage access goes through an adapter interface (put/delete/getPublicUrl/replace); **R2 is the confirmed implementation**, the boundary is retained for testability and exit (§13.3.8) | MUST | P2 | `AC-038` |
| `FR-102` | Object keys are random and opaque; never derived from original filename or sequential | MUST | P2 | `AC-039` |
| `FR-103` | Content-type validated against magic bytes; mismatches rejected | MUST | P2 | `AC-037` |
| `FR-104` | Per-file size limit enforced server-side with a clear Indonesian error | MUST | P2 | `AC-037` |
| `FR-105` | EXIF (including GPS) stripped on upload | MUST | P2 | `AC-040` |
| `FR-106` | Responsive width variants referenced via `srcset` with explicit dimensions. Variants are **pre-generated, produced client-side before upload, or delegated to Cloudflare Images** — never generated by `sharp` inside a Worker (`D-19`; §13.8) | SHOULD | P2 | `AC-040` |
| `FR-107` | Public absolute CDN URL stored in the database; never a local or deployment-internal path | MUST | P2 | `AC-038` |
| `FR-108` | Media library grid with search, type filter, pagination, and metadata display | MUST | P3 | `AC-041` |
| `FR-109` | Deletion checks references, warns, requires confirmation, then removes the object and row | MUST | P3 | `AC-042` |
| `FR-110` | Alt text required for content images; enforced at publish | MUST | P3 | `AC-041` |
| `FR-111` | Media replacement preserves the media record and existing references | SHOULD | P3 | `AC-043` |
| `FR-112` | Failed uploads leave no orphan database row and no partial object | SHOULD | P2 | `AC-042` |
| `FR-113` | Delivery uses optimised formats with fallbacks and lazy loading below the fold; format conversion may rely on Cloudflare Images or pre-generated files (`D-19`) | SHOULD | P5 | `AC-040` |
| `FR-114` | Accepted formats restricted (jpeg/png/webp/avif); SVG rejected for user uploads | MUST | P2 | `AC-037` |
| `FR-115` | Oversized images are downscaled to a maximum dimension **before** storage (client-side or external step) or rejected; the Worker does not resize (`D-19`) | SHOULD | P2 | `AC-040` |
| `FR-116` | Media picker available inside product and article editors | MUST | P3 | `AC-041` |
| `FR-117` | Duplicate detection by checksum to avoid re-uploading identical files | COULD | P3 | `AC-039` |
| `FR-118` | Upload progress and failure feedback in Bahasa Indonesia | SHOULD | P3 | `AC-042` |

### 7.13 Functional requirements — settings & branding (`FR-120`–`FR-139`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-120` | Site identity settings (name, tagline, short description) editable and rendered site-wide | MUST | P3 | `AC-044` |
| `FR-121` | Contact & WhatsApp settings propagate to header, footer, `/kontak`, CTAs without a redeploy | MUST | P3 | `AC-045` |
| `FR-122` | WhatsApp number normalised to international form (leading `0` → `62`, non-digits removed) | MUST | P3 | `AC-005` |
| `FR-123` | Brand colour tokens editable with format validation and a contrast warning for failing combinations | SHOULD | P3 | `AC-046` |
| `FR-124` | Logo slot assignment (primary, mono, reversed, icon) via media references; empty slot renders a text fallback | MUST | P3 | `AC-044` |
| `FR-125` | Settings and content saves **purge the affected cached public URLs** so changes appear without a redeploy (explicit purge, not ISR-style revalidation — §12.4) | MUST | P3 | `AC-047` |
| `FR-126` | Preview of branding changes before saving | COULD | P3 | `AC-046` |
| `FR-127` | No secrets (API keys, DB URLs, tokens) in settings UI, settings table, or client bundles | MUST | P3 | `AC-048` |
| `FR-128` | Settings changes recorded in the audit log | SHOULD | P3 | `AC-035` |
| `FR-129` | SEO defaults configurable: title template, default meta description, default OG image | MUST | P5 | `AC-049` |
| `FR-130` | Social links configurable and rendered only when set | SHOULD | P3 | `AC-045` |
| `FR-131` | Operating hours configurable per day | SHOULD | P3 | `AC-045` |
| `FR-132` | Maps URL configurable; embed rendered only when valid | SHOULD | P3 | `AC-045` |
| `FR-133` | Settings validation with field-level Indonesian errors; invalid values are never persisted | MUST | P3 | `AC-046` |
| `FR-134` | Reset-to-default for individual settings groups | COULD | P3 | `AC-046` |
| `FR-136` | Favicon and app-icon upload slots with derived sizes | SHOULD | P3 | `AC-044` |
| `FR-137` | Constrained typography settings (no arbitrary font imports) | COULD | P3 | `AC-046` |
| `FR-139` | "Last saved" timestamp visible on settings screens | SHOULD | P3 | `AC-048` |

---

## 8. Publishing workflows

### 8.1 Status model

| Status | Public visibility | Indexable | Editable | Notes |
|---|---|---|---|---|
| `draft` | No (preview only) | No | Yes | Default for new records |
| `published` | Yes | Yes | Yes | Requires passing validation (`FR-162`) |
| `unpublished` | No (404) | No | Yes | Previously published, deliberately taken down |
| `archived` | No (404) | No | Yes (restorable) | Retired content, hidden from all lists |

Permitted transitions: `draft → published`, `published → unpublished`, `published → archived`, `unpublished → published`, `unpublished → archived`, `archived → draft`. Illegal transitions are rejected server-side, not merely hidden in the UI (`FR-170`).

### 8.2 Draft → preview → publish

1. Save as draft. Drafts are never reachable through the normal public route.
2. Preview renders the real page template behind a short-lived, signed token (`FR-161`). Preview responses send `X-Robots-Tag: noindex, nofollow` and are excluded from the sitemap.
3. Publish runs validation (required fields, slug uniqueness, price consistency, image alt text), then flips status and **purges** the affected cached public URLs (`FR-162`, `FR-179`).

### 8.3 Unpublish and archive

- Unpublish → the public URL returns 404 (not a "coming soon" page), removed from the sitemap, `noindex` if any cache remains.
- Archive → same public effect; record retained for history and possible restore.
- Neither action deletes media; cleanup is explicit (`FR-109`).

### 8.4 Slug change → redirect

When a **published** record's slug changes:

1. The previous slug is written to `slug_history` (entity type, entity ID, old slug, timestamp) (`FR-165`).
2. The old public URL returns **HTTP 301** to the new canonical URL (`FR-150`).
3. The sitemap contains only the new slug.
4. Chains collapse: if `a → b` then `b → c`, `a` resolves directly to `c` in one hop (`FR-177`).
5. A slug may not be reused by a different record while its history is live — reuse would hijack an indexed URL (`FR-166`).
6. Reserved slugs (route segments such as `produk`, `blog`, `admin`, `api`, `_next`, `sitemap.xml`) are rejected.

### 8.5 Media replacement

Replacing an image keeps the media record and its references: the stored object is replaced and variants regenerated, so product/article pages need no edits (`FR-111`, `FR-167`). If the provider cannot replace in place, the adapter uploads a new key and atomically updates the media record's URL — the DB URL is the only reference point.

### 8.6 Functional requirements — publishing & redirects (`FR-160`–`FR-179`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-160` | Four-state status model (`draft`/`published`/`unpublished`/`archived`) applied to products, articles, and services | MUST | P3 | `AC-056` |
| `FR-161` | Short-lived signed preview URL for drafts, excluded from sitemap, `noindex` | MUST | P3 | `AC-053` |
| `FR-162` | Publish gate runs full validation and refuses to publish invalid records | MUST | P3 | `AC-057` |
| `FR-163` | Unpublish removes the page from public reach (404) and from the sitemap | MUST | P3 | `AC-057` |
| `FR-164` | Archive hides the record from all lists but keeps it restorable | MUST | P3 | `AC-057` |
| `FR-165` | `slug_history` records every previous slug of a published record | MUST | P3 | `AC-054` |
| `FR-166` | Slug reuse by another record is blocked while history is live; reserved slugs rejected | MUST | P3 | `AC-054` |
| `FR-167` | Media replacement works inside the publishing flow without breaking references | SHOULD | P3 | `AC-043` |
| `FR-168` | Unsaved-changes guard when leaving an editor | SHOULD | P3 | `AC-057` |
| `FR-169` | Field-level validation with specific Indonesian messages; invalid saves rejected | MUST | P3 | `AC-057` |
| `FR-170` | Illegal status transitions rejected server-side regardless of UI state | MUST | P3 | `AC-056` |
| `FR-176` | Redirect management screen to view and remove slug history entries | SHOULD | P3 | `AC-054` |
| `FR-177` | Redirect chains collapsed; loops detected and refused | SHOULD | P3 | `AC-054` |
| `FR-178` | Slug-change warning shown in the editor before saving | MUST | P3 | `AC-054` |
| `FR-179` | Publish/unpublish **purges** every affected cached URL: the detail page, its listing/archive pages, and the sitemap (§12.4) | MUST | P3 | `AC-047` |

### 8.7 Functional requirements — Cloudflare platform (`FR-180`–`FR-199`, added in v0.2)

These cover the runtime, database and storage operations introduced by the platform switch. They are new, not renumbered.

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-180` | The application runs on the Cloudflare Workers runtime (V8 isolate / `workerd`), not on Node.js, and deploys with `wrangler deploy`. Local development uses `wrangler dev` so the runtime matches production | MUST | P1 | `AC-061` |
| `FR-181` | Runtime compatibility is verified explicitly: `nodejs_compat` is declared with a pinned `compatibility_date`, and no dependency relies on a Node API that Workers does not provide | MUST | P1 | `AC-061` |
| `FR-182` | No production code path reads or writes the Worker filesystem for data or uploads. `node:fs` is not used as a data store; `/tmp` is treated as request-scoped scratch only (§12.2) | MUST | P2 | `AC-063` |
| `FR-183` | The database is accessed through the D1 binding (`env.DB`). No connection string, credentials, or `file:` URL appears in application configuration | MUST | P2 | `AC-062` |
| `FR-184` | Schema migrations live in a version-controlled `migrations/` directory and are applied with `wrangler d1 migrations apply`, separately for local, preview and production | MUST | P2 | `AC-062` |
| `FR-185` | Every multi-statement write that must be atomic is issued as a single `batch()` call — there are no interactive transactions (§13.2.2) | MUST | P2 | `AC-069` |
| `FR-186` | Publishing, unpublishing and settings changes purge the affected cached URLs explicitly. Admin responses are never publicly cached | MUST | P3 | `AC-072` |
| `FR-187` | Media uploads stream to R2 through the adapter; no upload is buffered whole in memory beyond the configured limit and none is written to the filesystem | MUST | P2 | `AC-064` |
| `FR-188` | Media is delivered from an R2 **custom domain** on the owner's zone. `r2.dev` URLs are never used in production content | MUST | P5 | `AC-064` |
| `FR-189` | Large uploads may use a presigned PUT to R2, issued only to an authenticated, CSRF-checked admin request, with a short expiry; the resulting object is verified server-side before a media row is trusted | SHOULD | P3 | `AC-064` |
| `FR-190` | The bucket CORS policy is configured per environment so browser-direct uploads work in preview and production, and is restricted to the admin origin | MUST | P2 | `AC-064` |
| `FR-191` | Deleting media deletes the R2 object and then the D1 row, in that order, and a scheduled reconciliation reports orphans in both directions (§13.3.6) | SHOULD | P3 | `AC-075` |
| `FR-192` | Secrets are supplied by `wrangler secret put` (production) and `.dev.vars` (local, gitignored). No secret appears in `wrangler.jsonc` `vars`, the repository, logs, or any client bundle | MUST | P1 | `AC-080` |
| `FR-193` | The application reads configuration from Workers **bindings**, not `process.env`, and fails fast with a clear message when a required binding is absent | MUST | P1 | `AC-061` |
| `FR-194` | Sessions are stored server-side in D1 and carried in a signed `HttpOnly` cookie; signing uses Web Crypto HMAC, not a Node-only crypto module (§11.1) | MUST | P2 | `AC-067` |
| `FR-195` | A separate preview/staging environment uses its **own** D1 database and R2 bucket; production data is never reachable from preview | MUST | P1 | `AC-073` |
| `FR-196` | Structured logs and error reporting are enabled (`observability`), covering auth events, publish actions, upload failures and platform errors, with no secrets or full PII | MUST | P2 | `AC-078` |
| `FR-197` | A scheduled logical export of D1 is written to R2 on a documented cadence, and a restore into a non-production environment has been performed and recorded | SHOULD | P6 | `AC-058`, `AC-074` |
| `FR-198` | Server-side image transformation is **not** implemented in a Worker. If transformation is needed it is delegated to Cloudflare Images/Resizing or performed before upload (`D-19`) | MUST | P2 | `AC-065` |
| `FR-199` | The deployment is reproducible from the repository: a pinned Wrangler version, a committed `wrangler.jsonc`, and a documented deploy command that requires no manual dashboard steps beyond secrets and one-time resource creation | SHOULD | P1 | `AC-061` |

---

## 9. SEO requirements

Principles:

1. **Real HTML.** Primary content exists in server-rendered HTML; crawlers must not need JavaScript.
2. **Per-page metadata** from CMS fields with documented defaults and a title template.
3. **One canonical per page**, pointing at the served URL, never at a redirect or a 404.
4. **Structured data only when true.** `Organization`/`LocalBusiness` requires confirmed name, address, phone, and hours — if any required datum is missing, the node is omitted rather than guessed. `Product` includes `offers` only when a real price exists; inquiry-only products emit no `offers`.
5. **Drafts never indexable** — status check at render time plus `X-Robots-Tag` on preview.
6. **Slug changes preserve equity** via 301 (`FR-150`).
7. **Accessible media** — every content image requires alt text.
8. **No ranking promises** anywhere in documentation, UI copy, or sales material.
9. Sitemap contains only indexable absolute URLs with honest `lastmod`.
10. `robots.txt` allows public content, disallows `/admin`, `/api`, and preview paths, and references the sitemap.

### 9.1 Functional requirements — SEO (`FR-140`–`FR-159`)

| ID | Requirement | Priority | Phase | Verification |
|---|---|---|---|---|
| `FR-140` | Unique `<title>` and meta description per indexable page from CMS fields or documented defaults | MUST | P5 | `AC-049` |
| `FR-141` | Exactly one absolute canonical URL per indexable page, matching the served URL | MUST | P5 | `AC-049` |
| `FR-142` | OG and Twitter card tags (`og:title`, `og:description`, `og:image`, `og:url`, `og:type`, card type) | MUST | P5 | `AC-050` |
| `FR-143` | Generated `sitemap.xml` containing only indexable absolute URLs | MUST | P5 | `AC-051` |
| `FR-144` | Generated `robots.txt` disallowing `/admin` and `/api`, referencing the sitemap | MUST | P5 | `AC-051` |
| `FR-145` | `Organization`/`LocalBusiness` JSON-LD emitted only when all required business data is confirmed | MUST | P5 | `AC-052` |
| `FR-146` | `Product` JSON-LD with `offers` only when a real price exists; no fabricated price | MUST | P5 | `AC-052` |
| `FR-147` | `BreadcrumbList` JSON-LD matching the visible breadcrumb | MUST | P5 | `AC-052` |
| `FR-148` | `BlogPosting` JSON-LD for published articles | MUST | P5 | `AC-052` |
| `FR-149` | Draft/preview responses carry `noindex` meta and `X-Robots-Tag` | MUST | P5 | `AC-053` |
| `FR-150` | Slug change on a published record produces a 301 from old URL to new | MUST | P5 | `AC-054` |
| `FR-151` | Alt-text audit surfaces content images missing alt text | MUST | P5 | `AC-055` |
| `FR-152` | No ranking, position, traffic, or "SEO guarantee" claims in UI copy or metadata | MUST | P5 | `AC-055` |
| `FR-153` | Default OG image used when a page has no specific image | SHOULD | P5 | `AC-050` |
| `FR-154` | Consistent trailing-slash policy with permanent redirect to the canonical form | COULD | P5 | `AC-055` |
| `FR-155` | 404 responses return HTTP 404 with `noindex`, never a 200 error page | MUST | P5 | `AC-055` |
| `FR-156` | Sitemap `lastmod` reflects real modification time | SHOULD | P5 | `AC-051` |
| `FR-157` | Search-console verification token configurable in settings | SHOULD | P5 | `AC-051` |
| `FR-158` | Human-readable slugs only; no IDs in public URLs | MUST | P4 | `AC-055` |
| `FR-159` | Paginated archives use self-referencing canonicals (no cross-page canonical) | COULD | P5 | `AC-055` |

---

## 10. Branding & media management

### 10.1 Logo assets present (verified dimensions)

| File | Dimensions | Transparency | Intended use | Notes |
|---|---|---|---|---|
| `logo/Garda-Jaya-CCTV-Main-Logo.png` | 931×268 | Yes | **Primary lockup** — website header, light backgrounds, documents | Horizontal mark + wordmark, ~3.47:1 |
| `logo/Garda-Jaya-CCTV-Main-Logo-Monochrome-Navy.png` | 575×200 | Yes | Single-colour navy lockup — print, low-colour surfaces | Not usable on navy/dark backgrounds |
| `logo/Garda-Jaya-CCTV-Main-Logo-Only.png` | 300×250 | Yes | **Icon mark** — favicon, app icon, social avatar, compact UI | Non-square (1.2:1); 300 px width limits large icon sizes |
| `logo/Garda Jaya CCTV Logo Presentation.png` | 1536×1024 | No (opaque white) | **Mockup / brand presentation sheet — NOT a web asset** | Shows 4 variants, including a reversed white-on-navy version that exists only as pixels inside this sheet |

### 10.2 Asset gaps

| Gap | Impact | Mitigation |
|---|---|---|
| **No vector master** (SVG/AI/EPS) | Logo cannot scale crisply; high-DPI and large-format use degrade | Request a vector from the designer; until then use PNG at ≤ native size |
| **No delivered reversed/white variant** | A white-on-navy logo appears in the presentation sheet but is **not** an available file; a Deep Navy header/footer cannot carry the logo correctly | Interim: keep header/footer on light surfaces or place the navy logo on a white plate. Do not extract from the mockup (§10.5) |
| **Icon is 300×250, non-square** | Favicons and app icons need square canvases; upscaling to 512 px blurs | Derive on a square transparent canvas with padding; request a 1024 px square export |
| **Monochrome lockup is navy only** | No white or black monochrome for single-colour print or dark UI | Request black and white monochrome masters |
| Presentation sheet is opaque and ~1.2 MB (1,209,659 bytes) | Unsuitable as a web asset | Never ship it; reference only |

### 10.3 Favicon and app-icon derivation

Derived from the icon mark at build/prep time and served from `public/` or media storage — **not** by editing `logo/`.

| Asset | Size | Notes |
|---|---|---|
| `favicon.ico` | 16/32/48 multi-size | ICO container |
| `icon-192.png` | 192×192 | Android/PWA |
| `icon-512.png` | 512×512 | Only if a non-upscaled source exists; otherwise document the limitation |
| `apple-touch-icon.png` | 180×180 | Opaque background (iOS handles transparency poorly) |
| Maskable icon | 512×512 with ≥20% safe padding | Only if source quality allows |

If the source cannot support 512 px without upscaling, ship 180/192 and record the limitation — do not upscale.

### 10.4 Logo slots in the CMS

| Slot | Expected asset | Where rendered |
|---|---|---|
| `logo_primary` | Primary lockup | Header (light), footer (light), documents |
| `logo_mono` | Monochrome navy | Print-oriented contexts, low-colour surfaces |
| `logo_reversed` | White-on-navy | **Slot reserved — asset missing** (§10.2) |
| `logo_icon` | Icon mark | Favicon/app-icon source, compact UI, social avatar |

The CMS stores a media reference per slot so swapping the logo requires no deploy (`FR-124`). Until a reversed asset exists, that slot stays empty and the UI must handle the absence gracefully.

### 10.5 Asset handling rules

- `logo/` is **source reference only**; never edit, rename, optimise in place, or delete.
- **Do not upscale** raster logos beyond native dimensions.
- **Do not auto-trace** to manufacture a vector; a traced approximation degrades the brand and creates a fake "master".
- Do not extract the reversed variant from the presentation mockup — request the real file.
- Production logo usage points at media-storage URLs managed by the CMS; the repo copy is the design reference.
- Logo alt text is the business name; decorative duplicates use empty alt.

---

## 11. Security & access requirements

| Control | Requirement |
|---|---|
| Role model | Single `Administrator` role. No RBAC, invitations, or teams (`FR-091`) |
| Password storage | Never reversible, never logged (`FR-081`). **Algorithm is Open (`OQ-015`):** Argon2id/bcrypt cost ≥ 12 were written for a Node.js runtime and may exceed the Workers CPU budget documented in §12.2 (the figure is not restated here). PBKDF2-SHA256 via Web Crypto is the leading candidate; it must be benchmarked on the real runtime before selection. Until `OQ-015` closes, the *property* (slow, salted, one-way, per-password salt) is mandatory and the *algorithm* is provisional |
| Session validation | **Every** admin request re-validates the session server-side. Middleware alone is insufficient and is treated as a UX redirect only (`FR-082`, `FR-092`) |
| Cookies | `httpOnly`, `Secure`, `SameSite=Lax`/`Strict`, scoped path; no tokens in `localStorage` (`FR-083`) |
| CSRF | Origin/`Sec-Fetch-Site` checks plus a per-session token on all state-changing requests (`FR-084`) |
| Login throttling | 5 failed attempts per 15 minutes per IP+account, progressive delay, generic message (`FR-085`) |
| Session lifetime | Absolute expiry (e.g. 7 days) and idle timeout (e.g. 2 hours); logout invalidates server-side (`FR-086`, `FR-095`) |
| Credential bootstrap | Account created from environment configuration or a one-time setup step; **no default credentials in the repository** (`FR-094`) |
| Input validation | Validate and normalise at every boundary: route params, query, form bodies, uploads, settings (`NFR-015`) |
| Output safety | Rich text sanitised server-side on save and again on render; no unsanitised HTML injection |
| Headers | CSP (report-only first, then enforce), `X-Content-Type-Options: nosniff`, `Referrer-Policy`, frame protection, HSTS via host (`NFR-013`) |
| Secrets | Environment variables only; `.env*` gitignored; never in CMS settings, client bundles, or logs (`FR-127`) |
| Error handling | Public errors are generic; details go to server logs. No stack traces to users |
| Admin exposure | `/admin` is `noindex`, disallowed in `robots.txt`, excluded from sitemap, never linked from public pages (`FR-097`) |
| Audit | Content and settings changes recorded with actor, action, entity, timestamp (`FR-093`, `FR-128`) |

Auth library: **Auth.js or a vetted session library is proposed, not committed** (`OQ-004`). A hand-rolled session layer must still satisfy every control above.

### 11.1 Session and secret mechanics on Workers

These are design constraints, not implementation:

| Concern | Direction | Status |
|---|---|---|
| Session transport | Signed `HttpOnly` cookie; HMAC signature via **Web Crypto** (`crypto.subtle`), no Node-only crypto module | Proposed |
| Session store | A `sessions` table in **D1**, keyed by a random opaque session ID; server-side revocation is a row delete | Proposed |
| External session store | Workers KV or a hosted session service would work, but **KV is outside the approved scope** — it would need a new owner decision (`OQ-016`) | Out of scope |
| Secrets | `wrangler secret put` for production; `.dev.vars` for local, gitignored; **never** in `wrangler.jsonc` `vars`, never in the repo, never in the client bundle | Confirmed as a rule |
| Admin gate (alternative) | **Cloudflare Access (Zero Trust)** can front `/admin` as a complementary identity gate before the app's own login. Cost/tier and whether it is warranted for a single-operator CMS must be verified — see `OQ-015` and `D-22`. **Cost exposure is `RISK-002` (included-limit ceilings); no risk entry currently models the admin-access model itself, which is why it stays an Open question rather than a documented risk** | Open |
| CSRF | Origin/`Sec-Fetch-Site` check plus a per-session token; unchanged from v0.1 (`FR-084`) | Confirmed |
| Rate limiting | Application-level counters in D1; Cloudflare WAF rate-limiting rules are a possible complement, to be verified before reliance | Proposed |

---

## 12. Technology & deployment constraints

### 12.1 Confirmed platform decisions (owner-mandated)

| Layer | Decision | Status |
|---|---|---|
| Hosting / runtime | **Cloudflare Workers** | **Confirmed** (owner) |
| Database | **Cloudflare D1** | **Confirmed** (owner) |
| Media / object storage | **Cloudflare R2** | **Confirmed** (owner) |
| Framework | Not decided — evaluated in §12.6–§12.9 | **Proposed — awaiting owner sign-off** |
| Language | TypeScript, `strict` | **Confirmed** |
| Data model | SQLite-compatible relational schema (D1 *is* SQLite) | **Confirmed** |
| Rendering | Server-rendered HTML for public pages, SEO-first; no client-only rendering of primary content | **Confirmed** |

The v0.1 target (Node.js on a Vercel host, Next.js, Turso/libSQL, provider-agnostic media) is **superseded**. See §1.4 and §13.7. This is not a refactor of that plan — it is a different platform, and several v0.1 requirements had to be amended rather than merely re-hosted (§13.8).

### 12.2 Workers runtime model — what actually changes

| Property | Reality on Workers | Consequence for this product |
|---|---|---|
| Execution model | V8 isolates on `workerd`, not a Node.js process. No long-lived process, no in-process state that survives a request | No in-memory caches as source of truth; sessions and data live in D1. `[verified: https://developers.cloudflare.com/workers/, 2026-10-26]` |
| Node.js APIs | Node built-in modules require the `nodejs_compat` compatibility flag. **The exact compatibility date at which `nodejs_compat` (and its v2 variant) becomes default, and the precise default behaviour, must be confirmed against the compatibility-flags reference at implementation time.** `[unverified — confirm before implementation]` | Node-targeting libraries may work, but "it imports" is not "it is supported". Each dependency is checked, not assumed |
| Filesystem | **Not a real disk.** `node:fs` maps to a memory-based Virtual File System: `/bundle` is read-only, `/tmp` is writable but **"the contents of `/tmp` are not persistent and are unique to each request"**. `[verified: https://developers.cloudflare.com/workers/runtime-apis/nodejs/fs/, 2026-10-26]` | A SQLite `file:` database and any upload written to disk are **impossible in production**, not merely discouraged. Uploads stream to R2; data lives in D1 |
| CPU time | 10 ms per request on Workers Free; 5 min maximum on Paid (default 30 s). Waiting on network I/O does **not** count. `[verified: https://developers.cloudflare.com/workers/platform/limits/, 2026-10-26]` | A hard design budget. No `sharp`, no native image processing, no heavy synchronous hashing inside a request. See §12.2.1 |
| Memory | 128 MB per isolate on both Free and Paid. `[verified: https://developers.cloudflare.com/workers/platform/limits/, 2026-10-26]` | Large multipart bodies must stream, not be buffered whole |
| Request body size | 100 MB on Free/Pro, 200 MB on Business (account plan, not Workers plan). `[verified: https://developers.cloudflare.com/workers/platform/limits/, 2026-10-26]` | Plenty for photos; still not a reason to proxy uploads through the Worker when a presigned PUT will do (§13.3) |
| Response | Documented as having **no enforced response-body limit**; streaming supported. `[verified: https://developers.cloudflare.com/workers/platform/limits/, 2026-10-26]` | Long pages and file downloads can stream |
| Outbound connections | **6** simultaneous outgoing connections per request, on both Free and Paid. `[verified: https://developers.cloudflare.com/workers/platform/limits/, 2026-10-26]` | Batch D1 work; avoid fan-out |
| Secrets | `wrangler secret put` / `.dev.vars`; bindings, not `process.env` | `process.env` reliance is a bug, not a style choice |

> **This table is the single authoritative source for Workers platform numbers in this document (v0.2.1).** Every other mention of a CPU, memory, request-size or connection figure — in §11, §12.2.1, §12.5, §13.4, §14, §16 and §17, and in `AGENTS.md` — either points back here or carries the same `[verified: …]` label. **If a number appears anywhere without one of those two treatments, it is a documentation defect.** Re-verify every figure against the linked page before implementation; these are volatile and none is a requirement in itself.

#### 12.2.1 The CPU budget is a product constraint, not an implementation detail

Two v0.1 requirements are **materially changed** by this limit and are reclassified as Open:

1. **Image optimisation (`NFR-019`, `FR-106`, `FR-113`, `FR-115`).** `sharp` cannot run in a Worker. Options: Cloudflare Images / Image Resizing (paid tiers, and the Free plan allows 5,000 unique transformations per month `[verified: https://developers.cloudflare.com/images/pricing/, 2026-10-26]`), client-side pre-resize before upload, or pre-generated variants produced at upload time by an external step. **Not solved here** — `D-19`, `OQ-014`.
2. **Password hashing (`FR-081`).** Argon2id with production-grade parameters is designed to be expensive; that is precisely what the Workers CPU limit documented in §12.2 punishes. PBKDF2-SHA256 via Web Crypto (`crypto.subtle`, see `https://developers.cloudflare.com/workers/runtime-apis/web-crypto/`) is the leading candidate, with a benchmark on the real runtime as the acceptance test. **Not solved here** — `OQ-015`.

A third, quieter consequence: **any** request path that does synchronous work over large data — rendering a 200 KB article body, parsing a huge multipart form, computing many checksums — must be measured, not assumed.

### 12.3 Static assets and rendering strategy

- Static assets (CSS, JS, fonts, images committed to the build) are served by the Workers **static assets** feature, not by application code. `[verified: https://developers.cloudflare.com/workers/static-assets/, 2026-10-26]`
- Public pages are **server-rendered on demand from D1** so the owner's edits appear without a deploy (`FR-125`, `FR-179`). Purely static pages (privacy policy shell, 404) may be pre-rendered at build time.
- Primary content must be in the HTML response. Crawlers must not need JavaScript (unchanged from v0.1 — `FR-015`).

### 12.4 Caching and invalidation — **there is no ISR**

The v0.1 document assumed incremental static regeneration, which was a property of the Vercel hosting target and does not exist here. It is **removed** (§13.7). The replacement is explicit and testable:

| Layer | Mechanism | Invalidation |
|---|---|---|
| HTML responses | `Cache-Control` with `s-maxage` + `stale-while-revalidate` on public pages; Cloudflare CDN caches at the edge | **Explicit purge by URL** on publish / unpublish / settings change (`FR-186`) |
| Static assets | Content-hashed filenames, immutable long-lived caching | New hash per build |
| R2 media | Public custom domain behind Cloudflare cache; long `Cache-Control` because keys are random and immutable | New key on replace; old key purged or left to expire |
| Per-request work | Cache API for expensive derived values (e.g. rendered nav) — optional, not a source of truth | Versioned key |

Rules:

1. **No page may be cached in a way that serves a draft, an unpublished record, or stale admin data.** Admin responses are `Cache-Control: no-store`.
2. A publish action must purge the affected URLs *and* the listing/archive/sitemap URLs, not just the detail page.
3. Cache correctness is verified by `AC-070` (draft never served from cache) and `AC-072` (edit visible after purge) — not by assumption.

### 12.5 Environments, domains and deployment

| Environment | Runtime | Data | Purpose |
|---|---|---|---|
| Local | `wrangler dev` (workerd locally) | Local D1 + local R2 emulation | Development; closest possible parity |
| Preview / staging | Workers (separate Worker or environment) | **Separate D1 database** and separate R2 bucket | Pre-production verification; never the production database |
| Production | Workers | Production D1 + R2 | Live site |

- **Domain & DNS:** the domain must be a zone in the same Cloudflare account as the R2 bucket for a bucket custom domain to work. `[verified: https://developers.cloudflare.com/r2/buckets/public-buckets/, 2026-10-26]` Ownership of account + zone is a dependency (`DEP-011`, `OQ-017`).
- **Region:** Workers run at the edge; D1 has a primary location. Data-location hints exist, but latency to Indonesian users and D1 primary placement must be measured, not assumed. `[unverified — confirm before implementation]` (`OQ-018`).
- **Plan tier:** Free vs Paid changes the CPU budget by orders of magnitude and changes D1 limits — the documented figures live in the §12.2 limits table and in §13.2.1, not here. This is a launch decision, not a billing detail (`OQ-018`, `RISK-016`).

### 12.6 Framework evaluation (v0.2) — method and evidence

**Why re-evaluate.** The v0.1 framework choice was justified partly by platform alignment with the old host ("first-class support, zero-config deploys"). That justification evaporates with the platform change, so the choice is reopened rather than inherited.

**Candidates (exactly three):**

| | Candidate | Version substantiated |
|---|---|---|
| A | **Astro** + `@astrojs/cloudflare` | `astro` and `@astrojs/cloudflare` — current major versions were checked against the npm registry during this phase; re-verify before implementation `[verified: https://registry.npmjs.org/astro/latest and https://registry.npmjs.org/@astrojs/cloudflare/latest, 2026-10-26]` |
| B | **React Router framework mode** + Cloudflare Vite plugin | `react-router` — current major version checked against the npm registry during this phase; re-verify before implementation `[verified: https://registry.npmjs.org/react-router/latest, 2026-10-26]` |
| C | **Next.js on Workers via `vinext`** | `vinext` (**beta**, per Cloudflare's guide) and `next` — current versions checked against the npm registry during this phase; re-verify before implementation `[verified: https://registry.npmjs.org/vinext/latest and https://registry.npmjs.org/next/latest, 2026-10-26]` |

> **Why no exact semver here (v0.2.1).** Earlier drafts pinned exact versions (`astro` 7.4.2, `react-router` 8.5.0, `vinext` 1.3.1, `next` 16.4.3, `@opennextjs/cloudflare` 1.20.13). Those numbers were re-checked and were accurate *on 2026-10-26*, but they are not requirements — they go stale within weeks and would silently mislead an implementer. They have been removed from the prose. The binding statement is the **major-version direction** (React Router is on v8, not "v7+"), which is what actually constrains the evaluation. Re-resolve exact versions from the registry at scaffold time.

**Which Next.js path does Cloudflare actually endorse?** Cloudflare's Next.js guide states, verbatim: *"Cloudflare recommends vinext as the default way to run Next.js applications on Cloudflare Workers."* and *"vinext is in beta."* The same page lists the **OpenNext adapter** as a secondary path, in a table row whose "Use when" cell reads: *"You maintain an existing OpenNext application that cannot yet migrate to vinext because of a compatibility gap."* `[verified: https://developers.cloudflare.com/workers/framework-guides/web-apps/nextjs/, 2026-10-26 — re-verified 2026-10-26 during the v0.2.1 correction pass]` So: **vinext is endorsed by Cloudflare and is beta; `@opennextjs/cloudflare` (1.20.13, npm `latest`) is documented but positioned as a migration path for existing OpenNext apps.** A beta endorsement is still a beta.

> **Provenance note (v0.2.1).** Both sentences above were fetched from the live page and matched word-for-word during this revision, and the npm `latest` versions in §12.6 were re-queried from the registry. Earlier drafts of this row paraphrased the OpenNext table cell with an added *"if you"*; that paraphrase has been removed so that everything inside quotation marks is literal. If either sentence is not found on the page at implementation time, treat this row as stale and re-derive it — do not carry it forward on trust.

**Evidence protocol used in this section.** Every platform claim is labelled either `[verified: <url>, <date>]` — fetched and read during this revision — or `[unverified — confirm before implementation]`. Volatile specifics (exact limits, pricing, beta status, major versions) are quoted with their source and dated, because they will drift. **No figure in this section is estimated or invented.** Where a claim could not be checked, it is marked unverified rather than asserted.

**What the scores are and are not.** The 1–5 scores below are a structured judgement against this product's requirements, anchored to the cited documentation. They are **not** benchmark results, and no performance number is claimed.

### 12.7 Comparison matrix (1 = poor fit, 5 = strong fit)

| # | Criterion | A. Astro | B. React Router v8 | C. Next.js + vinext |
|---|---|---|---|---|
| 1 | **SEO**: metadata, sitemap, canonical, structured data, SSR/SSG quality, `<head>` control | **5** — content-first; per-page `<head>` is the default mental model; static output by default | **4** — full SSR; but the Cloudflare guide states *"SPA mode and prerendering are not currently supported when using the Cloudflare Vite plugin"*, so no static generation | **4** — metadata API is strong; static generation supported; beta toolchain in the path |
| 2 | **Public-site performance** (JS shipped, caching, image handling) | **5** — zero JS by default; islands only where needed | **3** — React hydration for the whole app shell | **3** — heaviest baseline; image optimisation only *"partially supported"* per Cloudflare |
| 3 | **Admin ergonomics**: auth, CRUD, forms, validation, upload UX | **3** — viable, but interactive admin is an island architecture inside a content framework; forms need deliberate design | **5** — loaders/actions, progressive enhancement, `useNavigation`, error boundaries are exactly a CRUD admin's shape | **4** — Server Actions and RSC are excellent for forms; constrained by beta status |
| 4 | **D1 integration + migrations** (adapter maturity, local parity, transactions) | **4** — bindings via `locals.runtime.env`; D1 is driver-agnostic SQL either way | **4** — bindings via `context.cloudflare.env` in loaders/actions; same SQL | **3** — bindings via `cloudflare:workers`; additional abstraction layer, and Drizzle/vinext interplay is less travelled |
| 5 | **R2 uploads** (validation, access control, presigned/direct) | **3** — API routes can do it; direct-to-R2 needs custom endpoints | **4** — actions handle multipart naturally; presigned flow is plain server code | **4** — route handlers/Server Actions handle uploads well |
| 6 | **TypeScript support & maintainability** | **5** — typed frontmatter/content collections, strict-friendly | **4** — good types; framework churn across majors has been real | **4** — excellent types; the reimplementation layer is an extra surface to debug |
| 7 | **Free commercial-use admin template availability** | **3** — most admin templates assume a React SPA shell; fewer Astro-native options | **4** — the React admin-template ecosystem applies directly | **4** — same ecosystem, plus Next-specific dashboards |
| 8 | **Runtime compatibility / Workers limits** | **5** — official adapter, `output: 'server'`, `prerender = true` per route | **5** — first-class: the Cloudflare Vite plugin runs server code in the Workers runtime locally | **4** — Cloudflare's own feature table marks App Router, RSC, Server Actions, SSR, static generation, ISR, middleware and **Cloudflare bindings** as "Supported"; only *"`next/*` imports: Mostly supported"* and *"Image optimization: Partially supported"* fall short. Raised from 3 in v0.2.1 on that evidence |
| 9 | **Local dev, testing, deploy, ops** | **5** — `wrangler deploy` auto-detects; C3 scaffold | **5** — `wrangler deploy` auto-detects; Vite plugin gives prod-like local runtime | **3** — `vinext check`, `vinext init`, separate `dev:vinext`/`build:vinext` scripts; more moving parts |
| 10 | **Long-term maintenance risk** (maturity, beta dependence) | **5** — stable, widely deployed, adapter maintained by Cloudflare | **4** — mature, but v8 is a young major | **2** — depends on a **beta** reimplementation of another vendor's framework API |
| | **Total (max 50)** | **43** | **42** | **35** |

### 12.8 Candidate narratives

**A. Astro + `@astrojs/cloudflare` — strongest fit for this product's centre of gravity.**
Astro is built for exactly this shape: *"designed for creating websites that display large amounts of content (such as blogs, documentation sites, or online stores)"*, shipping *"minimal client-side JavaScript"*, with *"islands"* only where interactivity is needed. `[verified: https://developers.cloudflare.com/workers/framework-guides/web-apps/astro/, 2026-10-26]` On Workers it is a first-class citizen: `wrangler deploy` detects Astro and generates the config, the adapter sets `output: 'server'`, and static routes opt out per page with `export const prerender = true`. `[verified: https://developers.cloudflare.com/workers/framework-guides/web-apps/astro/, 2026-10-26]`

The honest weakness is criterion 3. A form-heavy admin inside a content framework means islands, and island boundaries are where complexity hides. That is a real cost — but it is paid once, in the admin, and the admin is used by **one** person. The public site, which is the commercial asset and the SEO surface, gets the framework's full strength.

**B. React Router v8 + Cloudflare Vite plugin — the runner-up, and genuinely close.**
Cloudflare describes it as *"a full-stack React framework"* that *"combines with the Cloudflare Vite plugin to provide a first-class experience"*, and the plugin *"runs your server code in the Workers runtime, ensuring your local development environment is as close to production as possible"*. `[verified: https://developers.cloudflare.com/workers/framework-guides/web-apps/react-router/, 2026-10-26]` Loaders/actions are the best-in-class answer to admin CRUD, validation round-trips and optimistic UI. If the admin were the dominant surface, this would win outright.

Two documented caveats: prerendering and SPA mode are not supported with the Cloudflare Vite plugin (so the site is SSR-only), and the React admin-template ecosystem applies, which helps criterion 7.

**C. Next.js via `vinext` — capable, and the riskiest.**
`vinext` is *"a Vite plugin that reimplements the Next.js API surface"*, supporting App Router, RSC, Server Actions, SSR with streaming, and static generation; *"`next/*` imports: Mostly supported"*; image optimisation *"partially supported"*. `[verified: https://developers.cloudflare.com/workers/framework-guides/web-apps/nextjs/, 2026-10-26]` It is also explicitly **beta**, with a compatibility dashboard and a `vinext check` step that Cloudflare tells you to run before adopting it for production. Adopting it means betting a small business's CMS on a reimplementation of a third party's API, where a Next.js release can outrun the reimplementation.

**What was corrected here in v0.2.1, and what was not.** This candidate's identity was re-checked against the live documentation and **stands as written**: `vinext` is what Cloudflare's Next.js guide recommends, it is described there as beta, and the OpenNext adapter is the documented fallback. An earlier audit suspected these quotations were fabricated; they are not, and the provenance note in §12.6 records the verification. What *did* change is one score. Criterion 8 (runtime compatibility) was set to **3** on a reading of the guide that underweighted its own feature table — that table marks App Router, RSC, Server Actions, SSR with streaming, static generation, ISR, middleware **and Cloudflare bindings** as "Supported", reserving "mostly"/"partially" for `next/*` imports and image optimisation only. **3 → 4** is the defensible correction.

Criterion 10 (long-term maintenance risk) stays at **2**, deliberately, and the audit's suggestion to raise it to 3 was **not** adopted. The reason is concrete rather than conservative: the guide itself says *"vinext is in beta"* and instructs you to run a compatibility check *"before adopting it for existing production application[s]"*. A beta reimplementation of another vendor's API is the textbook definition of maintenance risk for a small business with one operator and no in-house engineering. Moving it to 3 would make the number match a preferred narrative rather than the evidence.

The audit also suggested recasting candidate C entirely as `@opennextjs/cloudflare`. That was **rejected**, because the documentation does not support it: Cloudflare positions OpenNext as the path for teams *"already on OpenNext"*, not as the recommended default. Rewriting candidate C as OpenNext would have made the document less accurate in the name of appearing more conservative. The honest position is that the endorsed path is also the beta path — which is precisely why this candidate is ranked third.

### 12.9 Recommendation, runner-up and flip condition

> **Recommendation: Astro + `@astrojs/cloudflare`. Status: Proposed — awaiting owner sign-off.** Not confirmed. No implementation may begin on it until the owner records the decision (`OQ-013`) and the §18.1 Phase 0 gate passes.

**Runner-up: React Router v8 framework mode + Cloudflare Vite plugin** (42 vs 43 — a one-point margin, well inside "defensible").

**Full ranking after the v0.2.1 re-score: Astro 43, React Router 42, Next.js + `vinext` 35 (max 50).** Candidate C's total moved from 34 to 35 on the single criterion-8 correction described in §12.8. **The recommendation does not change, and it does not change because the arithmetic does not move it:** a 35 sits 7–8 points behind both leaders, and the gap is not concentrated in one arguable criterion — it is spread across public-site performance (2), admin ergonomics (3), D1 integration (4 vs 4 vs 3), dev/ops (3) and maintenance risk (2). To invert the ranking on the evidence, candidate C would need to gain roughly 8 points, which no reading of the documented feature table supports.

**Why the margin is honest rather than decisive.** The two candidates tie or nearly tie on criteria 4, 8 and 9. The decision turns on which surface dominates. This product is content/marketing-dominant with a single-operator CMS, which favours Astro on criteria 1, 2 and 10. It is *not* a tie on 3, where React Router is clearly better.

**Exact evidence that would flip the decision to React Router:**

1. A working admin prototype in Astro shows that the island boundary forces either a duplicated data layer or an unacceptable amount of client-side JavaScript (measurable: admin route JS payload, and the number of endpoints duplicating validation).
2. The chosen free admin template passes the §7.10 licence gate but is a React SPA shell that cannot be decomposed into islands without forking it — collapsing criterion 7 for Astro to ≤ 2.
3. Cloudflare's Astro adapter develops a documented limitation affecting our routes (e.g. per-route `prerender` behaviour with D1), making criterion 8 fall below 4.

**What would flip the decision toward Next.js + `vinext`?** Nothing short of a structural change in this product's shape or in the adapter's status. Concretely: (a) Cloudflare removing the beta label **and** publishing a stable compatibility guarantee for `next/*` imports and image optimisation; (b) the admin growing into the dominant surface *and* a Next-specific admin template passing §7.10 where the React Router option does not; or (c) the owner mandating Next.js for reasons outside this document (existing team skills, a future migration to a Node host). Even then, the honest reading is that this would move candidate C toward the React Router runner-up rather than past Astro. **Do not treat a score movement alone as a flip condition** — a 35 becoming a 40 through re-scoring still trails, and re-scoring is not evidence.

Conversely, evidence that would confirm Astro beyond doubt: a public-page JS budget met with zero framework JavaScript shipped, and the admin prototype staying under a documented payload target.

**Rejected alternatives (briefly):**

| Alternative | Why not |
|---|---|
| **SvelteKit / Nuxt / SolidStart** | Viable on Workers, but a smaller hiring/maintenance pool for a small business, and no advantage on criteria 1–5 that justifies the risk |
| **Next.js + `@opennextjs/cloudflare`** | Real, documented and production-used, but Cloudflare's own guide now positions it as the path for teams *"already on OpenNext"* rather than the default. It carries genuine friction: partial feature support, larger bundles, longer cold starts, and a build step that can break on Next.js minor upgrades. Scored inside candidate C's rationale rather than as a fourth column — see §12.8 |
| **Next.js + `vinext`** | The documented default for Next.js on Workers, and therefore a legitimate candidate — but **beta**, per Cloudflare's own wording. Rejected as a *recommendation* for a one-operator CMS on maintenance-risk grounds (`RISK-014`), not on capability grounds |
| **Hono + hand-rolled frontend** | Maximum control, minimum magic — but we would rebuild routing, forms, and rendering that all three candidates provide, with no owner benefit |
| **WordPress** | Rejected in v0.1 and still rejected: bespoke brand, controlled stack, and the SQLite/D1 direction would be lost |
| **Static site generator only (no CMS runtime)** | Fails the core requirement: the owner must edit content without a developer or a deploy |
| **Cloudflare Pages** (instead of Workers) | Workers is the owner-confirmed runtime and now hosts static assets directly; Pages would add a second, converging platform for no gain |

### 12.10 What the framework decision does **not** change

Whatever is chosen, these hold: D1 for data, R2 for media, server-rendered public HTML, admin at `/admin` with server-side session validation, no ISR-style implicit caching, and the adapter boundaries in §13.3 so storage stays swappable.

---

## 13. Database & media persistence decisions

### 13.1 The three concepts (do not conflate)

1. **SQLite as the data model.** D1 *is* SQLite, so the v0.1 modelling decision survives intact: relational schema, SQL, no Postgres-only types, no stored procedures, no arrays. **Confirmed.**
2. **A local development database — but not a `file:` path in application config.** Under `wrangler dev`, D1 is emulated locally on disk by Wrangler and addressed through the **binding**, not through a connection string. The application code is identical in local and production; only the binding's backing store differs. A `file:./local.db` URL is a *dev-only* concept and must never appear in production configuration. **Confirmed as a constraint.**
3. **Production persistence: Cloudflare D1 (owner-confirmed).** Data lives in a managed D1 database, not on any filesystem. Redeploying the application cannot lose it, because the Worker holds no data. See §13.2.

### 13.2 Cloudflare D1 — production database

**Why D1 fits.** The v0.1 data model was already SQLite-compatible, so no schema redesign is needed. D1 is a managed SQLite database reachable from a Worker through a binding (`env.DB`), with no connection string, no connection pool, and no driver secret to leak. `[verified: https://developers.cloudflare.com/d1/worker-api/d1-database/, 2026-10-26]`

#### 13.2.1 Semantics that change the design

| Property | Documented behaviour | Design consequence |
|---|---|---|
| **Transactions** | D1 operates in **auto-commit**. `batch()` runs statements *"sequentially, non-concurrently"*, and *"if a statement in the sequence fails, then an error is returned for that specific statement, and it aborts or rolls back the entire sequence"* — i.e. batched statements **are** a SQL transaction. `[verified: https://developers.cloudflare.com/d1/worker-api/d1-database/, 2026-10-26]` | **There is no interactive transaction** (no `BEGIN` … hold … `COMMIT` across multiple round trips). Every multi-step write must be expressed as **one `batch()` call**. This is the single most important D1 constraint for the admin (§13.2.2) |
| Query concurrency | Each database is single-threaded and processes queries one at a time; throughput ≈ 1 / query duration | Prefer fewer, batched requests over many small ones `[verified: https://developers.cloudflare.com/d1/platform/limits/, 2026-10-26]` |
| Queries per invocation | 1,000 (Paid) / 50 (Free); **a `batch()` counts as one query**, not one per statement | Batching is both a correctness and a capacity strategy `[verified: same]` |
| Max SQL statement length | 100 KB | Long generated SQL must be split `[verified: same]` |
| Max bound parameters | 100 per query | Bulk inserts must be chunked `[verified: same]` |
| Max row / string / BLOB size | 2 MB | No giant JSON blobs in a single row; store large bodies as text within this bound `[verified: same]` |
| Max database size | 10 GB (Paid) / 500 MB (Free); the 10 GB cap **cannot be raised** | Far beyond this product's needs, but it is a hard ceiling to record, not a soft one `[verified: same]` |
| SQL query duration | 30 s (Cloudflare API limit) | Irrelevant for CMS queries; relevant for bulk data migrations `[verified: same]` |
| Simultaneous connections | 6 per Worker invocation | Do not fan out parallel D1 calls unnecessarily `[verified: same]` |
| Time Travel | 30 days (Paid) / 7 days (Free) point-in-time recovery | A real recovery capability — but **not** a substitute for a tested export (§13.2.5) `[verified: same]` |

> These figures are **documented limits as of 2026-10-26** and are quoted with their source. They are volatile. Re-verify against the linked pages at implementation time; do not hard-code them into copy, comments, or UI.

#### 13.2.2 The batch rule, applied to real admin operations

Every operation below **must** be a single `batch()`, or it is not atomic:

| Operation | Statements that must commit together |
|---|---|
| Publish a product | `UPDATE products SET status, published_at` → `INSERT INTO audit_log` → `INSERT INTO slug_history` (only if the slug changed) |
| Change a published slug | `INSERT INTO slug_history (old_slug)` → `UPDATE products SET slug` → `INSERT INTO audit_log` |
| Unpublish / archive | `UPDATE products SET status` → `INSERT INTO audit_log` → purge-cache bookkeeping |
| Delete media with references | `DELETE FROM media` → `INSERT INTO audit_log` (object deletion in R2 is **not** transactional — see §13.3.6) |
| Create admin session | `INSERT INTO sessions` → `INSERT INTO audit_log` |

The **cache purge** and the **R2 object deletion** are deliberately outside the batch: they are not database operations and cannot participate in a SQL transaction. Their ordering and failure handling are specified in §13.3.6 (`FR-191`).

#### 13.2.3 Migrations

- Migrations are `.sql` files in a version-controlled **`migrations/`** directory. Wrangler records applied migrations in a `d1_migrations` table. `[verified: https://developers.cloudflare.com/d1/reference/migrations/, 2026-10-26]`
- Applied with `wrangler d1 migrations apply <database_name>` — `--local` for the local database, `--remote` for preview/production. `[verified: same]`
- **Never** auto-migrate on request. Migrations are an explicit, reviewed deploy step (`RISK-008` unchanged).
- If an ORM writes nested migration directories (Drizzle does), `migrations_pattern` must be configured; `wrangler d1 migrations create` only writes top-level files. `[verified: same]`
- Migration files are **append-only** once applied to any shared environment. Editing an applied migration is a defect, not a fix.
- `PRAGMA defer_foreign_keys = true` is available when a migration must violate FKs temporarily. `[verified: same]`

#### 13.2.4 Local development parity

`wrangler dev` runs the Worker in `workerd` locally with a local D1 instance, so the same binding, the same SQL, and the same dialect are exercised in development. `[verified: https://developers.cloudflare.com/d1/best-practices/local-development/, 2026-10-26]` Parity is therefore structural, not aspirational — but it is **not** identity: local D1 is a single-node emulation, so concurrency, replication lag and true latency must be validated in preview. Seed data is development-only and contains no fabricated business facts (§13.2.6).

#### 13.2.5 Backup, export and recovery — targets, not guarantees

| Mechanism | What it gives | Limit of the guarantee |
|---|---|---|
| **Time Travel** (point-in-time restore) | Restore to a timestamp within the retention window (30 days Paid / 7 days Free) `[verified: https://developers.cloudflare.com/d1/platform/limits/, 2026-10-26]` | Lives inside the same provider account; does not protect against account-level loss or an accidental destructive migration applied *and* aged out |
| **`wrangler d1 export`** | Logical `.sql` dump; supports `--table`, `--no-schema`, `--output` `[verified: https://developers.cloudflare.com/d1/best-practices/import-export-data/, 2026-10-26]` | *"A running export will block other database requests"* — schedule it off-peak `[verified: same]` |
| **Scheduled export to R2** | Off-platform-ish copy, versioned by date; Cloudflare documents a Workflows-based D1→R2 backup example `[verified: https://developers.cloudflare.com/workflows/examples/backup-d1/, 2026-10-26]` | Requires a Cron/Workflow; must be monitored, because a silently failing backup is worse than none |

**RPO ≤ 24 h / RTO ≤ 4 h (`NFR-008`) are *targets*, not contractual guarantees.** They are only credible once a restore has actually been performed into a non-production environment and recorded (`AC-058`). Retention length is an owner decision (`OQ-019`).

#### 13.2.6 Seed strategy

- Development seeds: clearly marked dummy records, loaded only into the local database.
- Production seeds: **none**. No demo products, no sample articles, no placeholder business facts (`FR-038`, `RISK-006`).
- The administrator account is created from a one-time setup step or a secret, never from a committed seed (`FR-094`).

### 13.3 Cloudflare R2 — media storage

R2 is the owner-confirmed media store. Buckets are **private by default**; public access is an explicit opt-in. `[verified: https://developers.cloudflare.com/r2/buckets/public-buckets/, 2026-10-26]`

#### 13.3.1 Access paths

| Path | Use | Notes |
|---|---|---|
| **Workers binding** (`env.MEDIA`) | Server-side put/get/delete from the admin | No credentials in code; preferred for all Worker-originated operations |
| **S3 API** | Presigned URLs, tooling, bulk operations | Requires an R2 API token (access key + secret) stored as a **secret**, never in `wrangler.jsonc` |
| **Public custom domain** | Delivery of published media to browsers | **Required for production.** `r2.dev` is for development only |
| **`r2.dev` subdomain** | Ad-hoc development | Cloudflare explicitly frames it as *"for non-production use cases"*; caching, WAF, Bot Management and Access are **not** available on it `[verified: same]` |

**Production media is served from a custom domain on the owner's zone, never from `r2.dev`.** This is a hard requirement (`FR-188`) because the `r2.dev` URL cannot be protected, cached, or governed like a production asset, and it would have to be migrated later anyway.

#### 13.3.2 Bucket layout and key convention

Keys are **random and opaque** (`FR-102`) — never derived from the filename, never sequential. A date/type prefix is permitted for operational sanity because it leaks nothing:

```
media/<yyyy>/<mm>/<uuid>.<ext>        original
media/<yyyy>/<mm>/<uuid>-<w>.<ext>    pre-generated variant (if the image strategy produces them)
```

`storage_key` and the resolved public `url` are both stored in D1 (`FR-107`). The URL is absolute and points at the custom domain.

#### 13.3.3 Upload flow

1. Admin authenticates and passes CSRF (`FR-084`).
2. Server validates: declared MIME, **magic bytes** (`FR-103`), size (`FR-104`), permitted format (`FR-114` — jpeg/png/webp/avif; **SVG rejected**).
3. Small files: the Worker streams the body to R2 via the binding.
4. Large files: the Worker issues a **presigned PUT** so the browser uploads directly to R2, bypassing the Worker's CPU and body handling. Presigned URLs grant one operation on one object with an expiry between 1 second and 7 days. `[verified: https://developers.cloudflare.com/r2/api/s3/presigned-urls/, 2026-10-26]` A presigned flow still requires server-side confirmation *after* upload before the media row is trusted (`FR-189`).
5. EXIF (including GPS) is stripped (`FR-105`) — see the caveat in §13.3.7.
6. The `media` row is inserted with key, URL, dimensions, size, alt text, checksum.

#### 13.3.4 CORS

Browser-direct uploads need a bucket CORS policy allowing the admin origin, the `PUT` method, and the required headers. CORS is only needed for browser interaction; server-side binding calls do not use it. `[verified: https://developers.cloudflare.com/r2/buckets/cors/, 2026-10-26]` CORS configuration is environment-specific and must be applied to both the preview and production buckets (`FR-190`).

#### 13.3.5 Delivery, caching and lifecycle

- Public media is cached at the Cloudflare edge via the custom domain. Long `Cache-Control` is safe because keys are immutable — a replacement produces a new key, never an overwrite of a cached URL.
- Cloudflare notes that *"by default, only certain file types are cached"* and that caching all files requires a Cache Everything rule. `[verified: https://developers.cloudflare.com/r2/buckets/public-buckets/, 2026-10-26]` This must be configured and verified, not assumed.
- Lifecycle rules may expire unreferenced objects after a retention window; orphan reconciliation (§13.3.6) is the primary control, lifecycle is the backstop.

#### 13.3.6 Deletion, orphans and the non-transactional boundary

Deleting a media row and deleting the R2 object are **two systems that cannot share a transaction**. The rule:

1. Delete the R2 object **first**, then the D1 row. If the D1 delete fails, the result is a row pointing at a missing object — detectable and repairable.
2. The reverse order would leave an unreferenced object — invisible and therefore worse.
3. A **reconciliation job** (scheduled) lists R2 keys and compares them against `media.storage_key`, reporting orphans both ways. This is a requirement (`FR-191`), not an optional nicety, precisely because the boundary is non-transactional.

#### 13.3.7 Known limits of this approach (stated honestly)

- **No `sharp`.** Server-side resizing is not viable (§12.2.1). Variants must be pre-generated, produced by an external step, produced client-side before upload, or delegated to Cloudflare Images. `[unverified — the chosen mechanism is an open decision]`
- **EXIF stripping without a native library** is non-trivial. A pure-JS EXIF parser can strip metadata blocks for JPEG, but coverage across formats must be tested; if a format cannot be stripped reliably, either reject it or document the residual risk. **Do not claim EXIF stripping works for a format that has not been tested.**
- **Magic-byte validation** must be implemented explicitly — a browser-supplied `Content-Type` is untrusted input.

#### 13.3.8 Storage adapter boundary (unchanged in intent, updated in target)

```
putObject({ key, body, contentType }) -> { url, key }
deleteObject({ key }) -> void
getPublicUrl({ key, variant? }) -> string
replaceObject({ key, body, contentType }) -> { url, key }   // may be implemented as put-new-key + update-row
```

Business logic depends on this interface, never on the R2 binding directly. R2 is confirmed as the implementation, but the boundary is retained so a future migration is one adapter plus a URL migration — and so tests can use an in-memory adapter. `replaceObject` is explicitly allowed to be implemented as *write a new key, then update the row atomically*, because R2 objects are immutable in practice once cached.

### 13.4 Decision table (the single decision log)

Status vocabulary: **Confirmed** = decided by the owner; **Proposed** = documented recommendation awaiting sign-off; **Open** = not decided, must not be built on; **Superseded** = recorded for history in §13.7, do not implement.

| ID | Area | Leading option | Status | Owner action |
|---|---|---|---|---|
| D-01 | Runtime | **Cloudflare Workers** (V8 isolates / `workerd`), TypeScript | **Confirmed** (owner) | — |
| D-02 | Framework | Astro + `@astrojs/cloudflare` (runner-up: React Router v8) — §12.9 | **Proposed** | Sign off (`OQ-013`) or choose runner-up |
| D-03 | Hosting | **Cloudflare Workers** | **Confirmed** (owner) | — |
| D-04 | Data model | SQLite-compatible relational schema | **Confirmed** | — |
| D-05 | Local dev DB | Wrangler's local D1 via binding; **no `file:` URL in production config** | **Confirmed** (constraint) | — |
| D-06 | Production DB | **Cloudflare D1** | **Confirmed** (owner) | — |
| D-07 | ORM / query layer | Drizzle ORM with D1 driver, or plain prepared statements — not yet chosen | **Proposed** | Sign off after a spike (`OQ-020`) |
| D-08 | Media storage | **Cloudflare R2** | **Confirmed** (owner) | — |
| D-09 | Admin template | None selected — evaluation only (§7.10) | **Open** | Approve shortlist, then one template (`OQ-020`) |
| D-10 | Auth library | Auth.js or vetted session library | **Proposed** | Sign off (`OQ-004`) |
| D-11 | Rich text editor | Tiptap / Lexical / Markdown+preview | **Open** | Approve after trial (`OQ-005`) |
| D-12 | Rich text storage | Sanitised HTML vs structured JSON | **Open** | Decide with editor choice (`OQ-005`) |
| D-13 | ~~Vercel region~~ | **Superseded** — replaced by D-20 | **Superseded** | See §13.7 |
| D-14 | Domain & DNS | Owner-provided domain, **zone on Cloudflare** | **Open** | Provide domain + account access (`OQ-017`) |
| D-15 | Backups | Time Travel + scheduled `d1 export` to R2 | **Proposed** | Approve retention + RPO/RTO (`OQ-019`) |
| D-16 | Analytics | None in MVP (optional privacy-friendly later) | **Open** | Confirm deferral (`OQ-012`) |
| D-17 | ~~Next.js App Router~~ | **Superseded** — framework re-opened in §12.6 | **Superseded** | See §13.7 |
| D-18 | ~~Turso (libSQL)~~ | **Superseded** — replaced by D-06 (D1) | **Superseded** | See §13.7 |
| D-19 | Image optimisation on Workers | Cloudflare Images/Resizing (paid tiers) vs client-side pre-resize vs pre-generated variants | **Open** | Decide before P2; `sharp` is not an option (`OQ-014`) |
| D-20 | D1 primary location / Workers placement | Closest to Indonesian users, measured not assumed | **Open** | Confirm after provisioning (`OQ-018`) |
| D-21 | Password hashing algorithm | PBKDF2-SHA256 via Web Crypto (leading) vs Argon2id vs Workers-compatible scrypt | **Open** | Benchmark, then approve (`OQ-015`) |
| D-22 | Admin access model | App login only, vs Cloudflare Access (Zero Trust) in front of `/admin`, vs both | **Open** | Decide tier/cost (`OQ-015`) |
| D-23 | Workers plan tier | Free vs Paid — the two tiers differ by orders of magnitude on CPU budget and D1 capacity. **Documented figures: §12.2 (Workers CPU/memory) and §13.2.1 (D1 limits); do not restate them here.** | **Open** | Decide before launch (`OQ-018`, `RISK-016`) |
| D-24 | R2 public delivery domain | Custom domain on the owner's zone (**required** for production) | **Open** | Provide hostname (`OQ-017`) |
| D-25 | ~~Vercel Blob / Cloudinary media~~ | **Superseded** — replaced by D-08 (R2) | **Superseded** | See §13.7 |
| D-26 | ~~ISR caching strategy~~ | **Superseded** — replaced by §12.4 | **Superseded** | See §13.7 |

### 13.5 Schema outline (proposed, SQLite-compatible)

| Table | Purpose | Key columns |
|---|---|---|
| `users` | Administrator account | `id`, `email`, `password_hash`, `role`, `created_at` |
| `sessions` | Server-side sessions | `id`, `user_id`, `expires_at`, `created_at`, `last_seen_at` |
| `product_categories` | Product categories | `id`, `slug`, `name`, `description`, `sort_order` |
| `products` | Products | `id`, `slug`, `name`, `category_id`, `brand`, `summary`, `description`, `specs` (JSON text), `price_display`, `price_amount`, `price_max`, `price_note`, `status`, `featured`, `seo_title`, `seo_description`, `published_at`, `created_at`, `updated_at` |
| `product_images` | Ordered product media | `id`, `product_id`, `media_id`, `sort_order`, `is_cover` |
| `post_categories` | Article categories | `id`, `slug`, `name` |
| `posts` | Articles | `id`, `slug`, `title`, `excerpt`, `body`, `cover_media_id`, `category_id`, `author_name`, `status`, `seo_title`, `seo_description`, `published_at`, `created_at`, `updated_at` |
| `services` | Layanan records | `id`, `slug`, `title`, `summary`, `body`, `process_steps` (JSON text), `image_media_id`, `sort_order`, `status` |
| `pages` | Static page content | `id`, `key`, `title`, `body`, `seo_title`, `seo_description`, `updated_at` |
| `media` | Uploaded assets | `id`, `storage_key`, `url`, `mime`, `width`, `height`, `size_bytes`, `alt_text`, `variants` (JSON text), `checksum`, `created_at` |
| `settings` | Key/value site configuration | `key`, `value` (JSON text), `updated_at` |
| `slug_history` | Redirect source of truth | `id`, `entity_type`, `entity_id`, `old_slug`, `created_at` |
| `inquiries` | Contact form submissions (if enabled) | `id`, `name`, `contact`, `message`, `product_id`, `created_at`, `ip_hash` |
| `audit_log` | Change trail | `id`, `actor_id`, `action`, `entity_type`, `entity_id`, `created_at`, `metadata` (JSON text) |

Constraints: integer primary keys or ULIDs (SQLite-friendly); timestamps stored in UTC and rendered in `Asia/Jakarta`; money stored as integer rupiah; no Postgres-specific types.

### 13.6 Migration, seed, dev/prod parity

- Migrations are versioned `.sql` files in `migrations/`, applied by Wrangler as an explicit step — **never** automatically on request in production (`RISK-008`, §13.2.3).
- Seed data contains **no fabricated business facts** and no fake products or articles. Development seeds may use clearly-marked dummy records, gated to development only.
- The administrator account is created from a one-time setup flow or a secret, never from a committed seed with a default password.
- Local and production run the **same schema, the same SQL and the same binding**; only the backing database differs (`NFR-017`). Local D1 is an emulation, so concurrency and latency must be exercised in preview, not local (`FR-195`).
- Data must survive redeployments and framework upgrades (`NFR-016`, `AC-058`) — on D1 this is structural, since the Worker holds no data.

### 13.7 Superseded decisions register (v0.1 → v0.2)

Retained deliberately. A future maintainer must be able to see what was decided, why it changed, and what replaced it — without resurrecting it.

| Superseded | v0.1 rationale | Why superseded | Replacement | Date |
|---|---|---|---|---|
| **Vercel hosting** (D-03, D-13) | First-class Next.js support, preview deploys, image optimisation | Owner decision: runtime is Cloudflare Workers | Workers (§12.2) | 2026-10-26 |
| **Turso / libSQL as production DB** (D-06, D-18) | SQLite-compatible over HTTP, usable from serverless functions | Owner decision: D1 is the database; D1 is SQLite natively, so no driver indirection is needed | Cloudflare D1 (§13.2) | 2026-10-26 |
| **Vercel Blob / Cloudinary / Supabase Storage** (D-08, D-25) | Provider-agnostic media with an adapter | Owner decision: R2 is the media store | Cloudflare R2 (§13.3) | 2026-10-26 |
| **ISR (Incremental Static Regeneration)** (D-26) | Vercel-specific revalidation of static pages | Does not exist on Workers; implicit background regeneration has no equivalent and no safe invalidation story | On-demand SSR + `Cache-Control` + explicit purge by URL (§12.4) | 2026-10-26 |
| **Next.js App Router as the framework** (D-02, D-17) | Platform alignment with Vercel; largest React ecosystem | The alignment argument died with the platform; re-evaluated on merits (§12.6–§12.9) | Framework is **Proposed**, not selected (`OQ-013`) | 2026-10-26 |
| **`@libsql/client` as the DB driver** (D-07) | Turso's client | Turso is no longer the database | D1 binding; ORM choice reopened (`OQ-020`) | 2026-10-26 |

**Do not reintroduce any of these without an owner-approved PRD revision.** A proposal that quietly reintroduces Vercel-shaped patterns — ISR, `@vercel/*`, Vercel Blob, `vercel.json`, an "edge runtime" configuration — is a documentation defect, not a technical preference.

### 13.8 v0.1 requirements amended (not deleted) by the platform switch

IDs are stable. The wording changed because the old wording described a platform this project no longer uses. Each amendment is listed so the change is auditable.

| Requirement (amended) | v0.1 wording (abridged) | v0.2 wording | Why |
|---|---|---|---|
| `FR-081` | "Argon2id (preferred) or bcrypt cost ≥ 12" | Property-based: slow, salted, one-way hash; **algorithm Open** (`OQ-015`) | Native-cost hashing may exceed the Workers CPU budget |
| `FR-101` | "provider-agnostic adapter" | Adapter retained; **R2 is the confirmed implementation** | Media provider is no longer open, but the boundary still earns its keep for testing and exit |
| `FR-106` | "responsive width variants generated" | Variants must be **pre-generated / external / client-side** — not generated by `sharp` in a Worker | No native image processing in a Worker (`D-19`) |
| `FR-113` | "optimised formats with fallbacks" | Unchanged in intent; delivery may rely on Cloudflare Images or pre-generated files | Same |
| `FR-115` | "oversized images downscaled on upload" | Downscaling happens **before** storage (client or external step) or is delegated | Same |
| `FR-125` | "trigger revalidation of affected public paths" | "purge the affected cached URLs" — explicit, not ISR-style revalidation | No ISR (§12.4) |
| `FR-179` | "revalidates all affected public paths" | "purges all affected cached URLs" | Same |
| `NFR-001` | Measured "on a Vercel preview URL" | Measured on a **Workers preview deployment** | Platform |
| `NFR-019` | "optimised formats with fallbacks, responsive images" with no statement of where transformation happens | Delivery attributes (formats with fallbacks, explicit dimensions, `srcset`/`sizes`, lazy loading) are in scope and testable; **on-Workers transformation is explicitly out of scope** and delegated to `D-19` | `sharp` cannot run in a Worker (§12.2.1) |

Full row text appears in place; this table is the change record, not a second source of truth. The amended rows above are the only places where an existing ID's wording changed; every other v0.1 requirement is carried forward verbatim.

---

## 14. Non-functional requirements

| ID | Requirement | Target / condition | Priority | Phase | Verification |
|---|---|---|---|---|---|
| `NFR-001` | Core Web Vitals on public pages | LCP ≤ 2.5 s, CLS < 0.1, INP < 200 ms at p75; Lighthouse mobile (4× CPU throttle, Slow 4G), production build on a **Workers preview deployment**, cold and warm cache | MUST | P5 | `AC-060` |
| `NFR-002` | Server response time | TTFB ≤ 800 ms for cached/static pages measured from Indonesia; uncached DB-backed pages ≤ 1.5 s p75 | SHOULD | P5 | `AC-060` |
| `NFR-003` | Accessibility | WCAG 2.1 AA: contrast ≥ 4.5:1 body text, ≥ 3:1 large text and UI boundaries; keyboard operable; visible focus; labelled forms; error identification | MUST | P5 | `AC-060` |
| `NFR-004` | Responsive behaviour | No horizontal scroll ≥ 320 px; layouts per §6.9; touch targets ≥ 44×44 px | MUST | P4 | `AC-060` |
| `NFR-005` | Browser support | Last 2 versions of Chrome, Edge, Firefox, Safari (desktop, iOS, Android) | SHOULD | P6 | Manual matrix |
| `NFR-006` | Semantic HTML & keyboard | Landmarks, one `h1` per page, logical heading order, skip link, focus management in dialogs/drawer | MUST | P5 | `AC-060` |
| `NFR-007` | Reliability | No unhandled 5xx on public pages under normal traffic; graceful error state on DB/provider outage | SHOULD | P6 | Fault-injection checklist |
| `NFR-008` | Backup / recovery | RPO ≤ 24 h, RTO ≤ 4 h (validated against provider capability); restore procedure documented and rehearsed once before launch | MUST | P6 | Restore drill record |
| `NFR-009` | Logging | Structured logs for auth events, publish actions, upload failures, provider errors; **no passwords, tokens, or full PII** | MUST | P2 | Log sample audit (`AC-035`) |
| `NFR-010` | Error visibility | Runtime errors surfaced to a monitoring channel; no silent failures | SHOULD | P6 | Alert test |
| `NFR-011` | Maintainability | TypeScript `strict`; small focused modules; no dead code; no commented-out blocks | MUST | P1 | Typecheck + review |
| `NFR-012` | i18n readiness | All user-facing strings centralised; `id-ID` and `Asia/Jakarta` as central constants; no multi-language UI at launch | SHOULD | P1 | String audit (`AC-007`) |
| `NFR-013` | Security headers | CSP (report-only → enforce), `nosniff`, `Referrer-Policy`, frame protection, HSTS via host | MUST | P5 | Header inspection |
| `NFR-014` | Dependency policy | Minimal dependencies; each addition justified, licence-checked, recorded; no unmaintained packages | MUST | P1 | Dependency review (`DEP-002`) |
| `NFR-015` | Validation at boundaries | Every external input validated and typed before use | MUST | P2 | Code review (`AC-028`) |
| `NFR-016` | Data durability across deploys | Content persists across redeploys, rollbacks, and cold starts; no production dependency on the deployment filesystem | MUST | P2 | `AC-058` |
| `NFR-017` | Dev/prod parity | Identical schema and query layer; only connection configuration differs | MUST | P2 | `AC-058` |
| `NFR-018` | Media durability | Uploaded media remains available after redeploys and is served from a CDN URL recorded in the DB | MUST | P2 | `AC-059` |
| `NFR-019` | Image optimisation | Modern formats with fallbacks, explicit dimensions, `srcset`/`sizes`, lazy loading below the fold | SHOULD | P5 | `AC-060` |
| `NFR-020` | Privacy & data minimisation | Collect only what the contact form needs; publish a privacy policy; no full PII in logs; hash IPs used for rate limiting | MUST | P4 | `AC-013` |
| `NFR-021` | General rate limiting | Public form and auth endpoints rate-limited; abuse returns 429 with a friendly message | SHOULD | P5 | `AC-031` |

**v0.2 additions — Workers-specific (`NFR-041`–`NFR-060`).** All targets below are **targets with stated test conditions**, not guarantees, and none is a fabricated benchmark.

| ID | Requirement | Target / condition | Priority | Phase | Verification |
|---|---|---|---|---|---|
| `NFR-041` | Cold-start expectation | No cold-start penalty separate from normal request latency is claimed or assumed; first-request latency is measured on a Workers preview deployment, not extrapolated from Node.js | MUST | P5 | `AC-076` |
| `NFR-042` | CPU-time budget as a design constraint | Every request path is profiled against the plan's CPU limit **as documented in the §12.2 limits table** (re-verify at implementation time; the figure is not restated here). Any path exceeding 50 % of the budget in the chosen tier is redesigned, not shipped | MUST | P2 | `AC-076` |
| `NFR-043` | Streaming responses | Public pages and file downloads stream rather than buffer, so time-to-first-byte is not gated on full render | SHOULD | P5 | `AC-076` |
| `NFR-044` | Cache hit ratio | ≥ 80 % of public HTML requests served from the Cloudflare edge cache **under a defined load test** (steady read traffic, purge events excluded). Reported with the test parameters, never as a bare percentage | SHOULD | P5 | `AC-072` |
| `NFR-045` | Cache correctness | A purge takes effect within 30 s of a publish action; a draft is never served from cache to an anonymous visitor | MUST | P3 | `AC-070`, `AC-072` |
| `NFR-046` | D1 query efficiency | Every public listing query uses an index and completes in < 50 ms p75 on the production dataset size; no unbounded table scan on a public route | MUST | P5 | `AC-076` |
| `NFR-047` | D1 write atomicity | No admin operation leaves a partially applied multi-table change; verified by fault injection into a `batch()` | MUST | P2 | `AC-069` |
| `NFR-048` | D1 capacity headroom | Storage and rows-read/written stay below 50 % of the plan's included limits at launch; the plan tier is revisited if projected usage crosses 70 % | SHOULD | P6 | `AC-077` |
| `NFR-049` | R2 delivery latency | Media served from the custom domain returns in < 300 ms p75 for a cache hit, measured from Indonesia | SHOULD | P5 | `AC-076` |
| `NFR-050` | Upload throughput | A 5 MB image uploads successfully over a simulated 3G connection without timing out the Worker; progress or failure is reported to the user | SHOULD | P3 | `AC-064` |
| `NFR-051` | Upload size ceiling | The enforced per-file limit is documented in the admin UI and consistent between client-side and server-side checks | MUST | P2 | `AC-064` |
| `NFR-052` | Secret hygiene | Zero secrets in the repository, in `wrangler.jsonc` `vars`, in logs, or in any client bundle; verified by automated scan in CI | MUST | P2 | `AC-080` |
| `NFR-053` | Bindings discipline | Configuration is read from bindings; `process.env` is not used for application configuration | MUST | P1 | `AC-061` |
| `NFR-054` | Environment isolation | Preview cannot read or write production D1 or R2; verified by attempting a cross-environment query and confirming failure | MUST | P1 | `AC-073` |
| `NFR-055` | Observability coverage | Auth failures, publish actions, upload failures and unhandled errors are queryable in Workers Logs with a correlation identifier | MUST | P2 | `AC-078` |
| `NFR-056` | Backup verifiability | A restore from an export is performed and recorded before launch; a backup that has never been restored is treated as unverified | MUST | P6 | `AC-074` |
| `NFR-057` | Deployment reproducibility | A clean checkout plus documented secrets can deploy a working build; no undocumented dashboard-only configuration is required | SHOULD | P1 | `AC-061` |
| `NFR-058` | Dependency Workers-compatibility | Every dependency is confirmed to run on Workers (or is replaced) before adoption; native addons and `fs`-dependent libraries are rejected | MUST | P1 | `AC-061` |
| `NFR-059` | Graceful platform degradation | A D1 or R2 failure produces a generic error page and a logged event, never a stack trace or a blank screen | SHOULD | P6 | `AC-078` |
| `NFR-060` | Cost predictability | A documented cost model exists for the chosen plan tier covering requests, CPU, D1 rows, R2 storage/operations and image transformations, with the free-tier ceiling stated explicitly | SHOULD | P6 | `AC-077` |

---

## 15. Acceptance criteria

Every criterion is independently verifiable and names the requirement(s) it verifies. Format is Given/When/Then for stateful behaviour and a checklist for static properties.

### 15.1 Coverage map

| AC | Verifies |
|---|---|
| `AC-001` | `FR-001`, `FR-002`, `FR-009`, `FR-011`, `FR-019` |
| `AC-002` | `FR-003`, `FR-004`, `FR-015` |
| `AC-003` | `FR-005`, `FR-013` |
| `AC-004` | `FR-006`, `FR-008`, `FR-012` |
| `AC-005` | `FR-007`, `FR-122` |
| `AC-006` | `FR-010`, `FR-018` |
| `AC-007` | `FR-014`, `FR-016`, `FR-017`, `NFR-012` |
| `AC-008` | `FR-020`, `FR-021`, `FR-022`, `FR-023`, `FR-039` |
| `AC-009` | `FR-024`, `FR-025`, `FR-038` |
| `AC-010` | `FR-026`, `FR-031`, `FR-033` |
| `AC-011` | `FR-027`, `FR-037` |
| `AC-012` | `FR-028`, `FR-032`, `FR-036` |
| `AC-013` | `FR-029`, `FR-034`, `FR-035`, `NFR-020` |
| `AC-014` | `FR-030` |
| `AC-015` | `FR-040`, `FR-044`, `FR-052` |
| `AC-016` | `FR-041`, `FR-042`, `FR-043`, `FR-059` |
| `AC-017` | `FR-045`, `FR-048`, `FR-049` |
| `AC-018` | `FR-046`, `FR-057` |
| `AC-019` | `FR-047` |
| `AC-020` | `FR-050`, `FR-051`, `FR-053`, `FR-054`, `FR-055`, `FR-056`, `FR-058` |
| `AC-021` | `FR-060`, `FR-061`, `FR-066`, `FR-069` |
| `AC-022` | `FR-062`, `FR-063`, `FR-067`, `FR-070` |
| `AC-023` | `FR-064`, `FR-068`, `FR-073`, `FR-075` |
| `AC-024` | `FR-065` |
| `AC-025` | `FR-071`, `FR-072`, `FR-074`, `FR-076`, `FR-077`, `FR-078` |
| `AC-026` | `FR-080`, `FR-098` |
| `AC-027` | `FR-081` |
| `AC-028` | `FR-082`, `FR-092`, `NFR-015` |
| `AC-029` | `FR-083` |
| `AC-030` | `FR-084` |
| `AC-031` | `FR-085`, `NFR-021` |
| `AC-032` | `FR-086`, `FR-095`, `FR-096` |
| `AC-033` | `FR-087`, `FR-088`, `FR-089`, `FR-090` |
| `AC-034` | `FR-091`, `FR-094` |
| `AC-035` | `FR-093`, `FR-128`, `NFR-009` |
| `AC-036` | `FR-097`, `FR-099`, `NFR-013` |
| `AC-037` | `FR-100`, `FR-103`, `FR-104`, `FR-114` |
| `AC-038` | `FR-101`, `FR-107` |
| `AC-039` | `FR-102`, `FR-117` |
| `AC-040` | `FR-105`, `FR-106`, `FR-113`, `FR-115` |
| `AC-041` | `FR-108`, `FR-110`, `FR-116` |
| `AC-042` | `FR-109`, `FR-112`, `FR-118` |
| `AC-043` | `FR-111`, `FR-167` |
| `AC-044` | `FR-120`, `FR-124`, `FR-136` |
| `AC-045` | `FR-121`, `FR-130`, `FR-131`, `FR-132` |
| `AC-046` | `FR-123`, `FR-126`, `FR-133`, `FR-134`, `FR-137` |
| `AC-047` | `FR-125`, `FR-179` |
| `AC-048` | `FR-127`, `FR-139` |
| `AC-049` | `FR-129`, `FR-140`, `FR-141` |
| `AC-050` | `FR-142`, `FR-153` |
| `AC-051` | `FR-143`, `FR-144`, `FR-156`, `FR-157` |
| `AC-052` | `FR-145`, `FR-146`, `FR-147`, `FR-148` |
| `AC-053` | `FR-149`, `FR-161` |
| `AC-054` | `FR-150`, `FR-165`, `FR-166`, `FR-176`, `FR-177`, `FR-178` |
| `AC-055` | `FR-151`, `FR-152`, `FR-154`, `FR-155`, `FR-158`, `FR-159` |
| `AC-056` | `FR-160`, `FR-170` |
| `AC-057` | `FR-162`, `FR-163`, `FR-164`, `FR-168`, `FR-169` |
| `AC-058` | `NFR-016`, `NFR-017`, `NFR-008` |
| `AC-059` | `NFR-018` |
| `AC-060` | `NFR-001`, `NFR-002`, `NFR-003`, `NFR-004`, `NFR-005`, `NFR-006`, `NFR-019` |
| `AC-061` | `FR-180`, `FR-181`, `FR-193`, `FR-199`, `NFR-053`, `NFR-057`, `NFR-058` |
| `AC-062` | `FR-183`, `FR-184` |
| `AC-063` | `FR-182`, `NFR-016` |
| `AC-064` | `FR-187`, `FR-188`, `FR-189`, `FR-190`, `NFR-050`, `NFR-051` |
| `AC-065` | `FR-198`, `FR-106`, `FR-115`, `NFR-019` |
| `AC-066` | `FR-103`, `FR-104`, `FR-114` |
| `AC-067` | `FR-194`, `FR-082`, `FR-092` |
| `AC-068` | `FR-046`, `FR-047` |
| `AC-069` | `FR-185`, `NFR-047` |
| `AC-070` | `FR-149`, `FR-186`, `NFR-045` |
| `AC-071` | `FR-143`, `FR-149`, `FR-165`, `FR-150` |
| `AC-072` | `FR-125`, `FR-179`, `FR-186`, `NFR-044`, `NFR-045` |
| `AC-073` | `FR-195`, `NFR-054` |
| `AC-074` | `FR-197`, `NFR-056` |
| `AC-075` | `FR-191`, `FR-109` |
| `AC-076` | `NFR-041`, `NFR-042`, `NFR-043`, `NFR-046`, `NFR-049` |
| `AC-077` | `NFR-048`, `NFR-060` |
| `AC-078` | `FR-196`, `NFR-055`, `NFR-059` |
| `AC-079` | `FR-024`, `FR-038`, `FR-152` |
| `AC-080` | `FR-192`, `FR-127`, `NFR-052` |

**Traceability note.** Every `FR-###` maps to at least one `AC-###`, and every `AC-###` names the `FR-###`/`NFR-###` it verifies. Four NFRs are **process-level** and are verified by the phase exit gates in §18.3 rather than by a single behavioural test: `NFR-007` (reliability), `NFR-010` (error visibility), `NFR-011` (maintainability), and `NFR-014` (dependency policy). They are not untested — they are checked by review and by the "no new unresolved security finding / typecheck and lint pass" exit criteria. When a concrete test is added for any of them, assign it a new `AC-###` from the reserved range and add a row above.

### 15.2 Criteria

**`AC-001`** (FR-001, FR-002, FR-009, FR-011, FR-019)
Given a public page at 360 px, when the user opens the navigation drawer, then it opens with `aria-expanded="true"`, traps focus, closes on `Esc`, and the footer shows the CMS-configured phone, email, and address plus a working WhatsApp action. The current page's nav item carries `aria-current="page"` and a visible style. Every `target="_blank"` link carries `rel="noopener noreferrer"`.

**`AC-002`** (FR-003, FR-004, FR-015)
Checklist: (a) every public page has `lang="id"`; (b) all visible public copy is Bahasa Indonesia with no language switcher; (c) `curl` of each public URL returns HTML containing that page's main heading text without executing JavaScript.

**`AC-003`** (FR-005, FR-013)
Given an unmatched path, the response is 404 with a Bahasa Indonesia page offering navigation home and a WhatsApp CTA. Checklist: no production page contains lorem ipsum, `TODO`, "Coming soon", or placeholder business claims.

**`AC-004`** (FR-006, FR-008, FR-012)
Given a category whose only product is a draft, that category is absent from navigation and listings. Inner pages show a breadcrumb whose links resolve. A mixed-case URL permanently redirects to the lowercase canonical form.

**`AC-005`** (FR-007, FR-122) — *WhatsApp link uses CMS number*
Given the admin sets the WhatsApp number `0812-3456-7890`, when any public page renders, then every WhatsApp CTA links to `https://wa.me/6281234567890?text=<urlencoded message>` (leading `0` normalised to `62`, non-digits removed), opens in a new tab with `rel="noopener noreferrer"`, and no copy states or implies the message is sent automatically.

**`AC-006`** (FR-010, FR-018)
Checklist: the skip link is the first focusable element and moves focus to main content; `header`/`nav`/`main`/`footer` landmarks exist; exactly one `h1` with no skipped heading levels. After a client-side navigation, focus lands in the main region.

**`AC-007`** (FR-014, FR-016, FR-017)
Checklist: a single central constant defines `id-ID` and `Asia/Jakarta` and all date/time rendering uses it; brand tokens are applied consistently across public pages; on mobile the sticky WhatsApp action never overlaps primary content or traps focus.

**`AC-008`** (FR-020, FR-021, FR-022, FR-023, FR-039)
Given homepage blocks configured in the CMS, when the homepage is published, then hero copy, service cards, featured products, and latest articles reflect CMS values on the next request without a redeploy. A block with no eligible content renders nothing — no empty container, no placeholder.

**`AC-009`** (FR-024, FR-025, FR-038)
Checklist: the trust strip contains only statements entered by the owner; no certification badge, testimonial, client logo, warranty period, project count, or years-of-experience claim appears anywhere in the build output; templates and seeds contain no prefilled business facts; the CTA band links to the CMS WhatsApp number.

**`AC-010`** (FR-026, FR-031, FR-033)
Given the owner edits the Tentang Kami body and the service-area text, when the page reloads, then the new copy renders and the service-area statement matches the owner's text exactly, with no invented coverage claims. No code change, commit, or redeploy was required.

**`AC-011`** (FR-027, FR-037)
Given at least two published services, `/layanan` renders cards linking to `/layanan/[slug]`; each detail page renders body, ordered process steps, the "what is included" list, and a working contact CTA. An unpublished service returns 404.

**`AC-012`** (FR-028, FR-032, FR-036)
Checklist on `/kontak`: phone is a `tel:` link, email a `mailto:` link, WhatsApp CTA works, hours render in `Asia/Jakarta` formatting, and the map embed appears only when a valid URL is configured. No internal configuration or stack information is exposed.

**`AC-013`** (FR-029, FR-034, FR-035)
Given the contact form, when a submission has an invalid contact field and an empty message, then field-level Bahasa Indonesia errors are returned and nothing is stored. A valid submission is stored once; an identical submission inside the rate-limit window returns 429. Checklist: success copy promises neither an automatic reply nor a specific response time; stored fields are limited to name, contact, message, optional product, and a hashed IP; no full PII appears in logs.

**`AC-014`** (FR-030)
Given the owner edits the privacy policy body, `/kebijakan-privasi` renders the new text, is linked in the footer, and appears in the sitemap.

**`AC-015`** (FR-040, FR-044, FR-052)
Given 13 published products and a page size of 12, `/produk` shows 12 cards with pagination and page 2 shows the remaining product; ordering is identical across reloads. With zero published products, an empty state with a WhatsApp CTA renders instead of a blank grid.

**`AC-016`** (FR-041, FR-042, FR-043, FR-059)
Given products across two categories and three brands, applying a category filter shows only matching products, the URL query reflects the filter, and browser back restores the previous result set. Brand filter, search, and sort behave the same way and combine correctly.

**`AC-017`** (FR-045, FR-048, FR-049)
Given a published product with three images and five spec rows, the gallery shows all images with the cover first, specs render as a label/value table in authored order, and the description is present in server-rendered HTML.

**`AC-018`** (FR-046, FR-057) — *Product without price renders correctly*
Given four products with `price_display` = `fixed`, `from`, `range`, `inquiry`, each detail page and catalog card renders `Rp 1.250.000`, `Mulai dari Rp 1.250.000`, `Rp 1.250.000 – Rp 2.000.000`, and `Hubungi kami untuk harga` respectively. Given a `fixed` product whose amount is null, publishing is blocked; if forced into the database, the renderer shows the inquiry text — never an empty string and never `Rp 0`. A price note, when set, renders adjacent to the price.

**`AC-019`** (FR-047)
Given a product detail page, activating "Tanya Harga via WhatsApp" opens a `wa.me` URL using the CMS number whose `text` parameter contains the product name and canonical product URL, URL-encoded.

**`AC-020`** (FR-050, FR-051, FR-053, FR-054, FR-055, FR-056, FR-058)
Related products appear, exclude the current product, and are all published. A draft product's URL returns 404 and it appears in no listing or sitemap. The product page's title, meta description, canonical, and OG tags match CMS fields. The category archive lists only published products. Product JSON-LD is valid and omits `offers` when no price exists. Publishing is blocked when any product image lacks alt text. Toggling the featured flag moves the product on the homepage without a deploy.

**`AC-021`** (FR-060, FR-061, FR-066, FR-069)
Given 13 published articles, `/blog` shows 12 with pagination and page 2 shows the last one; category filtering narrows the list and updates the URL; with zero published articles an empty state renders. A category archive lists only published articles.

**`AC-022`** (FR-062, FR-063, FR-067, FR-070)
Given a published article with `h2`/`h3` headings, the body HTML is present server-side, heading hierarchy is valid, author display name and publish date (formatted in `Asia/Jakarta`) are shown, and a table of contents links to real heading anchors when the article exceeds the length threshold.

**`AC-023`** (FR-064, FR-068, FR-073, FR-075)
Checklist: related articles exclude the current article and are all published; every body image has non-empty alt text taken from the media record; each published article emits valid `BlogPosting` JSON-LD; an article without a cover image falls back to the default OG image.

**`AC-024`** (FR-065)
Given a draft article, requesting its public URL returns 404 (not a rendered page, not a 200 with a banner), and the slug appears in no sitemap and no listing.

**`AC-025`** (FR-071, FR-072, FR-074, FR-076, FR-077, FR-078)
Checklist: reading time renders above the configured length threshold; share actions work without third-party trackers; the RSS feed validates and contains published articles only; an article saved without an excerpt derives one from the body; previous/next links stay within the same category; blog search filters by title/excerpt and persists in the URL.

**`AC-026`** (FR-080, FR-098)
Given `/admin` while logged out, the user is redirected to `/admin/masuk`; after a valid login the dashboard renders. Invalid credentials return a generic Bahasa Indonesia error that does not reveal whether the email exists.

**`AC-027`** (FR-081)
The `users` table contains only a password hash, never plaintext. The hash is a slow, salted, one-way function whose parameters are recorded with the algorithm name and version; the specific algorithm is whatever `OQ-015` resolves to after benchmarking on the Workers runtime. Each password uses a distinct salt. Correct password succeeds, wrong password fails, and no password appears in logs or error responses. Checklist: the chosen algorithm completes within the plan's CPU budget on the real runtime, and the recorded benchmark is attached to the decision.

**`AC-028`** (FR-082, FR-092) — *Admin route blocked when logged out*
Given no valid session cookie, requesting any `/admin/*` page or admin API route directly (e.g. via `curl`, bypassing the UI) returns a login redirect for pages and 401 for APIs — no admin data is returned. Given a session deleted server-side (logout elsewhere or expiry), a request carrying the stale cookie is still rejected. Removing middleware configuration must not by itself grant access, because every handler re-validates the session.

**`AC-029`** (FR-083)
Checklist: the session cookie carries `HttpOnly`, `Secure` in production, and `SameSite=Lax` or `Strict`; no session token or credential appears in `localStorage`, `sessionStorage`, or the client bundle.

**`AC-030`** (FR-084)
Given a state-changing admin request sent without a valid CSRF token or with a foreign `Origin`, the server rejects it with 403 and no data changes.

**`AC-031`** (FR-085)
Given 5 failed login attempts for the same IP+account within 15 minutes, a 6th attempt — even with the correct password — is rejected with 429 and a retry hint, and success is not granted until the window expires.

**`AC-032`** (FR-086, FR-095, FR-096)
Given a logged-in admin, logging out invalidates the session row server-side and the old cookie no longer grants access. An idle session beyond the configured timeout redirects to login on the next request. Changing the password invalidates other active sessions.

**`AC-033`** (FR-087, FR-088, FR-089, FR-090)
Given 3 draft and 2 published products, and 1 draft and 4 published articles, the dashboard shows exactly those counts. "Terakhir diperbarui" lists the 5 most recent updates with `Asia/Jakarta` timestamps. Quick links navigate to the correct screens. The admin layout has no horizontal scroll and reachable actions at 360 px.

**`AC-034`** (FR-091, FR-094)
Checklist: exactly one role exists (`administrator`); no role selector or permission matrix exists in UI or schema. No default password, seed credential, or hard-coded admin email exists in the repository; the account is provisioned from environment configuration or a one-time setup step.

**`AC-035`** (FR-093, FR-128, NFR-009)
Given an admin publishes a product and changes the WhatsApp number, `audit_log` contains two entries recording actor, action, entity, and UTC timestamp, and no entry contains a password, session token, or full PII. Log output for the same operations contains no credentials.

**`AC-036`** (FR-097, FR-099)
Checklist: `/admin` and all admin routes emit `noindex` and are disallowed in `robots.txt`; admin pages are absent from the sitemap; all admin labels and messages are in Bahasa Indonesia.

**`AC-037`** (FR-100, FR-103, FR-104, FR-114)
A valid JPEG under the size limit uploads successfully. A `.png` whose bytes are actually an HTML/PHP payload is rejected. A file above the size limit is rejected with a clear Bahasa Indonesia message and nothing is stored. An SVG upload is rejected.

**`AC-038`** (FR-101, FR-107)
Checklist: all storage access goes through the adapter interface; no page or component imports a provider SDK directly. Every `media.url` is an absolute public CDN URL; no row contains `file://`, a relative path, or a deployment-internal path.

**`AC-039`** (FR-102, FR-117)
Given two uploads of files both named `foto.jpg`, their stored keys differ, are random and opaque, and are not derivable from the filename or upload order; enumerating sequential keys reveals no other uploads. Re-uploading an identical file surfaces the existing asset rather than creating a duplicate.

**`AC-040`** (FR-105, FR-106, FR-113, FR-115)
A photo containing GPS EXIF metadata is stored with no EXIF/GPS fields. Configured width variants exist and are referenced via `srcset` with explicit dimensions. Delivery uses modern formats with fallbacks and lazy loading below the fold. An oversized image is downscaled to the configured maximum dimension.

**`AC-041`** (FR-108, FR-110, FR-116)
The media library paginates and filters and shows dimensions and size per asset. Saving a product or article with an image whose alt text is empty is blocked with a field-level error. A media picker is available inside both product and article editors.

**`AC-042`** (FR-109, FR-112, FR-118)
Attempting to delete media used by a published product shows a warning listing referencing records and requires explicit confirmation; after confirmation the object is removed from storage and the row deleted. A failed upload leaves no orphan row and no partial object. Upload failures display an Indonesian error message.

**`AC-043`** (FR-111, FR-167)
Given a product image referenced by a published product, replacing the file shows the new image after the affected cached URLs are purged, keeps the media record ID unchanged, and breaks no other page referencing that media.

**`AC-044`** (FR-120, FR-124, FR-136)
Uploading a new primary logo and saving settings updates the public header on the next request; the old asset is not auto-deleted; an empty logo slot renders a text fallback rather than a broken image. Favicon and app-icon slots accept uploads and derived sizes are served.

**`AC-045`** (FR-121, FR-130, FR-131, FR-132)
Updating phone, email, address, operating hours, social links, and maps URL updates header, footer, `/kontak`, and the WhatsApp CTA without a redeploy. A social link left empty renders nothing.

**`AC-046`** (FR-123, FR-126, FR-133, FR-134, FR-137)
Brand tokens accept only validated formats; invalid values are rejected with a message and never persisted; a preview shows the effect before saving; contrast warnings appear for combinations failing AA. Resetting a settings group restores documented defaults. Typography options are constrained to approved choices.

**`AC-047`** (FR-125, FR-179)
Given a settings or content change, saving **purges** the affected cached public URLs so the next request shows fresh content, while unaffected cached pages remain cached.

**`AC-048`** (FR-127, FR-139)
Checklist: no API key, database URL, or token field exists in the settings UI or `settings` table; a secret pasted into a content field is not echoed in public HTML; each settings screen shows a "last saved" timestamp.

**`AC-049`** (FR-129, FR-140, FR-141)
Checklist: each indexable page has a unique `<title>` and meta description sourced from CMS fields or documented defaults; each has exactly one canonical absolute URL matching the served URL; no canonical points to a 404, a redirect, or a query-string variant.

**`AC-050`** (FR-142, FR-153)
Checklist: indexable pages emit `og:title`, `og:description`, `og:image` (absolute), `og:url`, `og:type`, and Twitter card tags; when no page-specific image exists, the configured default OG image is used.

**`AC-051`** (FR-143, FR-144, FR-156, FR-157)
Checklist: `/sitemap.xml` is valid XML containing only indexable absolute URLs (no drafts, no `/admin`, no `/api`) with honest `lastmod`; `/robots.txt` disallows `/admin` and `/api` and references the sitemap by absolute URL; a configured verification token renders in the document head.

**`AC-052`** (FR-145, FR-146, FR-147, FR-148)
Given complete confirmed business data, the homepage emits a valid `Organization`/`LocalBusiness` node; given a missing required field (e.g. no address), the node is omitted entirely rather than emitted with placeholders. A priced product emits `Product` with `offers`; an inquiry-only product emits `Product` without `offers` and without a fabricated price. Inner pages emit `BreadcrumbList` matching the visible breadcrumb, and published articles emit `BlogPosting`. All JSON-LD parses without error in a validator.

**`AC-053`** (FR-149, FR-161) — *Draft is not indexable*
Given a draft record, its preview URL returns `X-Robots-Tag: noindex, nofollow` and a `noindex` meta tag, and the preview URL is absent from `sitemap.xml`. After publishing, the same URL serves the public page without `noindex`.

**`AC-054`** (FR-150, FR-165, FR-166, FR-176, FR-177, FR-178) — *Slug change 301s*
Given a published article at `/blog/tips-keamanan`, when the admin changes the slug to `/blog/tips-cctv-rumah` and saves, then (a) a `slug_history` row exists for the old slug; (b) `GET /blog/tips-keamanan` returns **301** with `Location: /blog/tips-cctv-rumah`; (c) the sitemap lists only the new URL; (d) after a second rename to `/blog/panduan-cctv`, the original URL redirects directly to the newest URL in one hop; (e) another record cannot claim `tips-keamanan` while the history is live; (f) the editor warns before the slug change is saved; (g) the redirect screen lists the entry and can remove it.

**`AC-055`** (FR-151, FR-152, FR-154, FR-155, FR-158, FR-159)
Checklist: an alt-text audit lists every content image lacking alt text; no screen or metadata claims guaranteed rankings, positions, or traffic; a non-canonical trailing-slash request permanently redirects to the canonical form; error pages return HTTP 404 with `noindex` rather than a 200; no public URL contains a numeric record ID; paginated archives use self-referencing canonicals.

**`AC-056`** (FR-160, FR-170)
Given a record in `archived` status, attempting a transition to `published` via the API is rejected with 409/422 and the status is unchanged. From `draft`, publishing is permitted. All illegal transitions are rejected server-side regardless of UI state.

**`AC-057`** (FR-162, FR-163, FR-164, FR-168, FR-169)
Given a published product, unpublishing makes the public URL return 404, removes it from listings and sitemap, and keeps it editable. Archiving hides it from default admin lists while remaining restorable. Re-publishing restores the same URL unless the slug changed, which triggers a redirect. Leaving an editor with unsaved changes prompts for confirmation. Publishing a product with a missing required field or `price_max < price_amount` is blocked with specific field errors and the status remains `draft`.

**`AC-058`** (NFR-016, NFR-017, NFR-008) - *Data survives a redeploy*
Given content created in production (a product, an article, a settings change), redeploying the application (new deployment, rollback, or cold start) leaves all records unchanged and readable. Checklist: no production configuration references a `file:` database URL; no `.db` file is committed or bundled; the deployed function's filesystem is never used for writes; development and production share one schema. Backup and recovery: a scheduled backup exists, the retention window matches the approved setting, and a restore into a non-production environment has been performed once and recorded before launch.

**`AC-059`** (NFR-018) — *Uploaded image survives a redeploy*
Given an image uploaded in production, after a redeploy in which the previous deployment is no longer active, the image URL still resolves from the CDN and the page renders it. Checklist: the media row stores an absolute CDN URL, and deleting a deployment breaks no published page.

**`AC-060`** (NFR-001, NFR-002, NFR-003, NFR-004, NFR-005, NFR-006, NFR-019)
Given a production build measured with Lighthouse mobile (4× CPU throttle, Slow 4G) on a **Workers preview deployment**: LCP ≤ 2.5 s, CLS < 0.1, INP < 200 ms, accessibility score ≥ 95, no horizontal scroll at 320/360/768/1024/1280 px. Checklist: skip link, landmarks, one `h1`, visible focus, labelled inputs, 44 px touch targets, `srcset` with explicit dimensions, lazy loading below the fold. The measurement environment, plan tier and date are recorded with the result.

---

**v0.2 acceptance criteria (`AC-061`–`AC-080`).** Each is objectively testable and names what it verifies.

**`AC-061`** (FR-180, FR-181, FR-193, FR-199, NFR-053, NFR-057, NFR-058) — *Runs on Workers, not Node*
Checklist: (a) `wrangler dev` starts the app and serves every public route locally; (b) the deployed build runs on the Workers runtime and returns correct HTML for `/`, `/produk`, `/blog`, `/kontak`; (c) `wrangler.jsonc` pins `compatibility_date` and declares `nodejs_compat`; (d) no application configuration is read from `process.env` — removing an expected binding produces the documented startup error, not `undefined`; (e) a clean checkout plus documented secrets deploys successfully with no undocumented dashboard step; (f) every runtime dependency is recorded as Workers-compatible.

**`AC-062`** (FR-183, FR-184) — *D1 migration from empty, idempotent on re-run*
Given an empty D1 database, running `wrangler d1 migrations apply <db> --local` creates the full schema with no error. Running the same command again applies **zero** migrations and exits successfully (idempotent). Checklist: no connection string, credential or `file:` URL exists in application configuration; the `migrations/` directory is version-controlled; each migration file is append-only; `d1_migrations` records the applied set.

**`AC-063`** (FR-182, NFR-016) — *Data survives redeploy*
Given a product, an article and a settings change created in the deployed environment, redeploying the application (new version, rollback, or a cold start) leaves all records unchanged and readable. Checklist: no production code path writes application data to the filesystem; no `.db`/`.sqlite` file is committed or bundled; a query run after redeploy returns the same rows.

**`AC-064`** (FR-187, FR-188, FR-189, FR-190, NFR-050, NFR-051) — *Image retrievable from R2 after redeploy*
Given an image uploaded through the admin, the resulting URL is on the R2 custom domain (not `r2.dev`) and returns the image with HTTP 200. After a redeploy in which the previous version is no longer active, the same URL still resolves and the page renders it. Checklist: the `media` row stores the absolute custom-domain URL and the storage key; CORS is configured for the admin origin in both environments; a presigned upload, when used, expires and is rejected after expiry; the documented size limit matches between client and server.

**`AC-065`** (FR-198, FR-106, FR-115, NFR-019) — *No server-side image processing in a Worker*
Checklist: no `sharp`, `jimp`, `canvas` or equivalent native/CPU-heavy image library is present in the deployed bundle; no request path performs pixel transformation; `srcset` references pre-existing variant keys or Cloudflare Images URLs; an oversized upload is either downscaled before reaching the Worker or rejected with a clear message.

**`AC-066`** (FR-103, FR-104, FR-114) — *Upload rejects bad MIME, bad magic bytes, oversize*
Given a valid JPEG under the limit, the upload succeeds. Given a file named `.png` whose bytes are an HTML/PHP payload, the upload is rejected on magic-byte mismatch. Given a file above the limit, the upload is rejected with a clear Bahasa Indonesia message and **no object and no row** are created. Given an SVG, the upload is rejected. In every rejection case, no partial object remains in R2.

**`AC-067`** (FR-194, FR-082, FR-092) — *Admin blocked server-side when unauthenticated*
Given no session cookie, `GET /admin`, `GET /admin/produk` and every admin API route return a redirect to `/admin/masuk` (pages) or **401** (APIs) — with no admin data in the body. This holds when the request is made with `curl`, bypassing the UI. Given a session row deleted server-side, a request carrying the old cookie is still rejected. Checklist: session signing uses Web Crypto HMAC; the cookie is `HttpOnly`, `Secure` in production, `SameSite=Lax`/`Strict`; no token is stored in `localStorage`/`sessionStorage`; removing route middleware does not grant access, because each handler re-validates.

**`AC-068`** (FR-046, FR-047) — *No price renders inquiry CTA, never "Rp 0"*
Given a product with `price_display = inquiry` and a null `price_amount`, the catalog card and detail page render `Hubungi kami untuk harga` and the "Tanya Harga via WhatsApp" CTA. The rendered HTML contains neither an empty price element nor the string `Rp 0`. Given a `fixed` product whose amount is null, the renderer falls back to the inquiry text and publishing is blocked. The WhatsApp `text` parameter contains the product name and canonical URL, URL-encoded.

**`AC-069`** (FR-185, NFR-047) — *Atomic batched writes*
Given a publish action that must update the record, insert an audit row and insert a slug-history row, the operation is issued as one `batch()`. With a fault injected into the third statement, the first two are rolled back: the record's status, the audit log and `slug_history` all show the pre-operation state. Checklist: no code path issues `BEGIN`, holds a transaction open across awaits, or performs a multi-table write as separate awaited calls.

**`AC-070`** (FR-149, FR-186, NFR-045) — *Draft returns noindex and is absent from sitemap*
Given a draft record, its preview response carries `X-Robots-Tag: noindex, nofollow` and a `noindex` meta tag, and the URL does not appear in `/sitemap.xml`. A request for the draft's public URL returns 404 and never a cached published page. After publishing, the same URL serves the page **without** `noindex` and appears in the sitemap. Checklist: no cache entry can serve a draft body to an anonymous visitor.

**`AC-071`** (FR-143, FR-149, FR-165, FR-150) — *Publish → slug change → old URL 301s*
Given a published article at `/blog/tips-keamanan`, when the admin changes the slug to `/blog/tips-cctv-rumah` and publishes, then (a) a `slug_history` row exists for the old slug; (b) `GET /blog/tips-keamanan` returns **301** with `Location: /blog/tips-cctv-rumah`; (c) the sitemap contains only the new URL; (d) a second rename redirects the original URL directly to the newest URL in one hop; (e) another record cannot claim the old slug while history is live. Checklist: the redirect survives a redeploy and is not dependent on an in-memory map.

**`AC-072`** (FR-125, FR-179, FR-186, NFR-044, NFR-045) — *Edit visible after purge*
Given a cached public page, the admin changes its content and saves. Within 30 s, a fresh request returns the new content, while an unrelated cached page remains cached (verified by a cache-status header). A publish action purges the detail page, the listing page and the sitemap. Checklist: admin responses carry `Cache-Control: no-store`; the cache-hit-ratio target in `NFR-044` is reported together with its load-test parameters.

**`AC-073`** (FR-195, NFR-054) — *Preview cannot reach production data*
Checklist: the preview environment is bound to a different D1 database ID and a different R2 bucket than production; a query issued from preview against a production-only identifier fails; no production secret is present in the preview environment; a record created in preview is absent from production.

**`AC-074`** (FR-197, NFR-056) — *Restore rehearsed*
Given a scheduled export has produced a dump, restoring that dump into a non-production database completes without error and the restored row counts match the source at export time. Checklist: the restore date, operator and result are recorded; retention matches the approved setting; a backup with no recorded restore is marked unverified.

**`AC-075`** (FR-191, FR-109) — *Deletion and orphan reconciliation*
Given media referenced by a published product, deletion warns with the referencing records and requires confirmation. On confirmation the R2 object is removed **first**, then the row. If the row delete is forced to fail, the orphan is reported by the reconciliation job rather than silently lost. Checklist: a deliberately orphaned object appears in the reconciliation report; a row pointing at a missing object also appears.

**`AC-076`** (NFR-041, NFR-042, NFR-043, NFR-046, NFR-049) — *Performance measured, not assumed*
Checklist, all measured on a Workers preview deployment on the target plan tier with the parameters recorded: (a) p75 CPU time per request path is recorded and no path exceeds 50 % of the tier's budget; (b) public listing queries are index-backed and complete under the `NFR-046` target; (c) responses stream (first byte arrives before full render completes); (d) media from the custom domain meets the `NFR-049` target for a cache hit; (e) cold-start behaviour is described from measurement, not extrapolation.

**`AC-077`** (NFR-048, NFR-060) — *Capacity and cost model*
Checklist: a written cost model lists requests, CPU, D1 rows read/written, D1 storage, R2 storage/operations and any image transformations, with the free-tier ceiling stated; projected launch usage is below 50 % of included limits; the trigger for revisiting the plan tier (70 % projection) is documented.

**`AC-078`** (FR-196, NFR-055, NFR-059) — *Observability and graceful failure*
Given a failed login, a publish action and a simulated upload failure, each produces a queryable log entry with a correlation identifier and no secret or full PII. Given a simulated D1 outage, a public page returns a generic Bahasa Indonesia error page with a 5xx status, not a stack trace or a blank screen. Checklist: `observability` is enabled in the Worker configuration; errors are not swallowed silently.

**`AC-079`** (FR-024, FR-038, FR-152) — *No fabricated trust claims*
Checklist: the build output contains no testimonial, certification badge, client logo, award, warranty period, project count or years-of-experience claim that the owner did not supply; no template, seed or default copy pre-fills such a claim; no page promises rankings, positions or traffic.

**`AC-080`** (FR-192, FR-127, NFR-052) — *No secret in the client bundle*
Checklist: a search of the built client assets finds no API token, database identifier used as a credential, session signing key, or R2 secret; `wrangler.jsonc` `vars` contains no secret value; `.dev.vars` is gitignored and absent from the repository; an automated scan in CI fails the build if a secret pattern is detected.

---

## 16. Dependencies & risks

### 16.1 Dependencies

| ID | Dependency | Impact if unavailable | Mitigation |
|---|---|---|---|
| `DEP-001` | Owner-supplied business facts (address, phone, hours, service area, real product data) | Cannot publish trust-building content or `LocalBusiness` JSON-LD; site launches with thin content | Collect via a structured content checklist before P4; publish only confirmed data; omit rather than invent |
| `DEP-002` | **Cloudflare account with Workers enabled**, plus Wrangler authenticated locally | No deployment target | Provision before P1; document required secrets in a `.dev.vars.example` once it exists (`OQ-017`) |
| `DEP-003` | **D1 database** provisioned by the owner (production + preview) | No production persistence; app cannot launch | Owner provisions before P2; `OQ-017` |
| `DEP-004` | **R2 bucket** + custom domain attached, provisioned by the owner | No media uploads; no product or article images | Owner provisions before P2; `OQ-017` |
| `DEP-005` | **Domain name + Cloudflare zone ownership** | No canonical production URLs; no R2 custom domain (which requires the zone in the same account) | Resolve `OQ-017` before P2; SEO work is provisional until then |
| `DEP-006` | Admin template licence verification | Admin UI cannot ship on an unverified licence | Complete §7.10 gate in P3; fall back to a custom minimal shell |
| `DEP-007` | Vector logo master and reversed/white variant from the designer | Brand cannot be applied to dark surfaces; favicons limited | Track as a brand task; use documented interim treatment (§10.2) |
| `DEP-008` | Rich text editor selection and storage format | Article and product bodies cannot be authored | Resolve `OQ-005` in P3; start with a minimal editor if delayed |
| `DEP-009` | **Framework decision** (`OQ-013`) | No scaffold can be chosen; Phase 1 cannot start | Close the gate in §18.1 before any scaffold work |
| `DEP-010` | Owner availability for content entry and review | Launch slips; placeholder content risk | Schedule content entry in P6 with a written checklist |
| `DEP-011` | **Cloudflare account + zone ownership** (same account for bucket custom domain) | Cannot attach an R2 custom domain; cannot manage DNS; production delivery blocked | Owner must confirm ownership and access; `OQ-017`, `RISK-013` |
| `DEP-012` | **Image-optimisation decision** (`D-19`, `OQ-014`) | Cannot finalise upload pipeline or `srcset` strategy | Decide before P2; interim: pre-generated variants only |
| `DEP-013` | **Password-hashing decision** (`D-21`, `OQ-015`) | Auth cannot be implemented to a fixed spec | Benchmark PBKDF2/Argon2id on Workers, then approve |
| `DEP-014` | **Admin access model decision** (`D-22`, `OQ-015`) — app login vs Cloudflare Access | Affects auth scope, cost and threat model | Decide with `OQ-015`; app login is mandatory regardless |
| `DEP-015` | **Workers plan tier** (`D-23`, `OQ-018`) | CPU budget and D1 limits differ by an order of magnitude between Free and Paid | Decide before performance work in P5; `RISK-016` |
| `DEP-016` | **Admin template licence verification** (`D-09`, `OQ-020`) — carries forward `DEP-006` scope | Admin UI cannot ship on an unverified licence | Complete §7.10 gate in P3 |

### 16.2 Risks

| ID | Risk | Impact | Mitigation |
|---|---|---|---|
| `RISK-001` | **Cloudflare vendor concentration** — runtime, database, storage and CDN from one vendor | Terms, pricing or limit changes affect the whole stack at once | SQL portability (D1 is SQLite, schema is plain SQL); R2 adapter boundary; documented `d1 export` path; no Cloudflare-proprietary SQL in business logic (`RISK-017`) |
| `RISK-002` | **Free-tier / included-limit ceilings** on Workers requests, CPU, D1 rows, R2 operations and image transformations | Site degrades or stops at a traffic spike; silent cost jump | Monitor usage; treat all limits as "re-verify at implementation"; decide the plan tier deliberately (`D-23`, `RISK-016`) |
| `RISK-003` | A `file:` database URL or a bundled `.db` reaching production | Silent data loss / writes that vanish per request | Hard rule in `AGENTS.md`; no `file:` URL in application config; `AC-062`, `AC-063` |
| `RISK-004` | Uploads written to the Worker filesystem (`/tmp` looks writable) | Images vanish when the request ends — and `/tmp` is per-request by design | Storage adapter only; `FR-182`; `AC-063`, `AC-064` |
| `RISK-005` | **Admin template licence risk** — public repo mistaken for commercial-use permission | Legal exposure and forced rework | Read the `LICENSE` file for every candidate; record findings; no template without a recorded licence decision |
| `RISK-006` | **Fabricated trust claims** (testimonials, certifications, years, counts) | Reputational and legal harm; misleading advertising | Editorial rule (§6.7); review before launch; no prefilled claims in templates or seeds |
| `RISK-007` | **Unknown contact details** at build time | Placeholder phone/address published | Treat as blocking for launch; hide unset fields; never invent data |
| `RISK-008` | **Silent schema migration** on deploy | Data corruption or downtime | Explicit migration step; no auto-migrate on request; tested against a restored copy |
| `RISK-009` | Admin UI leaking into public pages (shared CSS/JS) | Public site looks like a dashboard; performance and brand damage | Separate layouts and style boundaries; audit public bundles for admin code |
| `RISK-010` | **Scope creep toward e-commerce** (cart, checkout, stock, quotes) | Timeline blowout; architectural churn | Non-goals are explicit (§4.2); any change requires a PRD revision and owner approval |

**v0.2 additions (`RISK-011`–`RISK-018`).**

| ID | Risk | Impact | Mitigation |
|---|---|---|---|
| `RISK-011` | **Workers CPU limit exceeded** — SSR, hashing, uploads or large payload parsing blow the Free-tier budget documented in §12.2 | 1102 errors in production; login or publishing fails intermittently | Treat CPU as a design constraint (`NFR-042`); profile every path; decide the plan tier early (`D-23`); move heavy work off the request path |
| `RISK-012` | **D1 has no interactive transactions** and a multi-step admin write is implemented as separate awaits | Partial writes: a published record with no audit row, or a slug change with no redirect | Mandatory `batch()` for atomic writes (`FR-185`); fault-injection test (`AC-069`); code review checklist |
| `RISK-013` | **R2 public access / custom domain not configured**, or the domain is not in the same Cloudflare account | Media unreachable in production, or shipped from `r2.dev` and needing migration later | Owner confirms zone ownership (`DEP-011`); custom domain is a hard requirement (`FR-188`); `AC-064` |
| `RISK-014` | **Next.js-on-Workers adapter risk** — *not* a generic "beta tooling" risk. Two distinct exposures: (a) **adapter feature-coverage drift** — `vinext` is a beta reimplementation of the Next.js API surface, so a Next.js minor release can ship a feature the adapter does not cover (`next/*` imports are only *"mostly supported"*, image optimisation only *"partially supported"*); (b) **build breakage on Next.js upgrades** — the adapter's build step is version-coupled, so a routine `next` bump can fail the deploy rather than degrade gracefully | A feature the CMS needs is unavailable in the adapter, or an upgrade blocks shipping; work lands on a solo operator with no in-house engineering | Framework recommendation avoids this path entirely — Astro is Proposed (§12.9). If the owner overrides and picks candidate C: pin `next` exactly, run `vinext check` in CI on every bump, and record a documented exit path back to the runner-up before writing application code |
| `RISK-015` | **Cloudflare account / zone ownership ambiguity** | Nobody can attach the domain, rotate secrets, or restore a backup at 2 a.m. | Named owner + documented access; owner-held credentials; `OQ-017` |
| `RISK-016` | **Free-tier CPU ceiling** chosen by default and discovered late — the figure is in the §12.2 limits table | The whole admin (hashing, rich text save) may be infeasible on Free | Decide `D-23` before P2; benchmark hashing on the real tier (`OQ-015`) |
| `RISK-017` | **Vendor lock-in** (see `RISK-001`) — exit cost grows quietly | Migration becomes impractical after data and URLs accumulate | Adapter boundaries; plain-SQL schema; export cadence (`FR-197`); URLs stored as keys + base, not hard-coded hostnames |
| `RISK-018` | **Image-optimisation cost** — Cloudflare Images charges per transformation, and the Free plan caps unique transformations | Bill shock or broken images once the cap is hit | Decide `D-19` before building the pipeline; pre-generate variants where possible; monitor transformation counts (`NFR-060`) |

---

## 17. Open questions requiring owner approval

| ID | Question | Why it matters | Default assumption if unanswered |
|---|---|---|---|
| ~~`OQ-001`~~ | ~~Approve Next.js (App Router) as the framework?~~ — **Resolved / Superseded 2026-10-26.** The question was tied to the Vercel target and is replaced by `OQ-013`, which re-opens the framework on the merits. Next.js is now one of three evaluated candidates and is **not** selected (§12.9). | — | — |
| ~~`OQ-002`~~ | ~~Approve Turso (libSQL) as the production database?~~ — **Resolved 2026-10-26.** Owner decided on **Cloudflare D1**; recorded as `D-06` **Confirmed** and registered as superseded in §13.7. | — | — |
| ~~`OQ-003`~~ | ~~Which media storage provider?~~ — **Resolved 2026-10-26.** Owner decided on **Cloudflare R2**; recorded as `D-08` **Confirmed**. Remaining sub-decisions (custom domain, image optimisation) are `OQ-014` and `OQ-017`. | — | — |
| `OQ-004` | Approve **Auth.js** or a vetted session library, vs. a hand-rolled session layer? | Security risk surface and maintenance | Prefer the vetted library; if unanswered, a hand-rolled implementation must satisfy §11 in full |
| `OQ-005` | Store rich text as **sanitised HTML** or **structured JSON**? | Editor choice, rendering, migration flexibility | Sanitised HTML with a strict allowlist, re-sanitised at render |
| `OQ-006` | Which **admin template** after licence verification? | Legal exposure and design consistency | Build a minimal custom admin shell with no third-party template |
| ~~`OQ-007`~~ | ~~Preferred Vercel region and domain name?~~ — **Resolved / Superseded 2026-10-26.** The Vercel region question no longer applies. Replaced by `OQ-017` (domain, zone, account) and `OQ-018` (D1 primary location / Workers placement). | — | — |
| `OQ-008` | Approve **Kebijakan Privasi** wording; is counsel review required? | Legal accuracy and UU PDP context | Publish a plain-language draft marked for owner review; make no compliance claims |
| `OQ-009` | Is the **admin UI language** Bahasa Indonesia? | Usability for the non-technical owner | Bahasa Indonesia for all admin labels |
| `OQ-010` | Enable the **contact form**, or WhatsApp-only? | Scope, spam handling, data storage, privacy obligations | Contact form enabled and stored; disable if the owner prefers WhatsApp-only |
| `OQ-011` | Approve **backup retention** and the RPO/RTO targets in `NFR-008`? | Data-loss exposure | Daily logical backup, 7-day retention; RPO 24 h / RTO 4 h |
| `OQ-012` | Confirm **no analytics** in MVP, or approve a privacy-friendly option later? | Scope and privacy policy content | No analytics in MVP |

**v0.2 additions (`OQ-013`–`OQ-020`).** None of these is answered. They are the blockers between this document and implementation (§18.1).

| ID | Question | Why it matters | Default if unanswered |
|---|---|---|---|
| `OQ-013` | **Framework sign-off:** approve **Astro + `@astrojs/cloudflare`** (§12.9), or choose the runner-up **React Router v8**? | Everything downstream — scaffold, routing, admin shell, template compatibility — depends on it | **Blocked.** Do not scaffold either. No default is assumed |
| `OQ-014` | **Image optimisation approach:** Cloudflare Images/Resizing (paid tiers), client-side pre-resize, external pre-generation, or ship originals? | Cost, CPU, upload UX, `srcset` strategy | Pre-generated variants only; no on-the-fly transformation (`D-19`) |
| `OQ-015` | **Password hashing + admin access model:** PBKDF2-SHA256 vs Argon2id vs Workers scrypt; and is **Cloudflare Access (Zero Trust)** required in front of `/admin`, and at what tier/cost? | Auth security and feasibility under the CPU budget; cost of an extra gate | PBKDF2-SHA256 benchmarked first; app login only, Access deferred (`D-21`, `D-22`) |
| `OQ-016` | Is a **KV namespace** permitted as a session/rate-limit store, or must D1 carry it? | KV is outside the approved scope but is the conventional Workers session store | **D1 only.** KV requires explicit approval; do not add it |
| `OQ-017` | **Cloudflare account, zone and domain:** who owns the account, is the domain an active zone in it, and what hostname serves R2 media? | R2 custom domain requires the zone in the same account; DNS and TLS depend on it | Blocked for production delivery (`DEP-011`, `RISK-013`) |
| `OQ-018` | **Workers plan tier** (Free vs Paid) and **D1 primary location**? | CPU budget and D1 capacity differ by orders of magnitude between tiers (documented in §12.2 and §13.2.1); latency to Indonesia | Assume **Free** for early development only; must be decided before performance work |
| `OQ-019` | **D1 backup retention** and confirmation of the RPO ≤ 24 h / RTO ≤ 4 h *targets*? | Data-loss exposure; Time Travel window is 30 days Paid / 7 days Free | Daily export to R2, 7-day retention, targets accepted as targets only |
| `OQ-020` | **Admin template choice** after the §7.10 licence gate, and the **ORM/query layer** (Drizzle vs prepared statements)? | Legal exposure, design consistency, migration tooling | Minimal custom admin shell; plain prepared statements with `batch()` |

---

## 18. Phased delivery

### 18.1 Phases

| Phase | Name | Goal | Exit criteria |
|---|---|---|---|
| **P0** | Documentation & architecture approval | Agree scope, platform, framework, persistence, and the open questions that gate implementation | **This PRD (v0.2) and `AGENTS.md` reviewed by the owner, with every gate item in §18.1.1 satisfied.** Note: approval of the *document* is not approval to build — the gate is separate |
| **P1** | Foundation | Runnable skeleton on the confirmed platform | Worker runs under `wrangler dev` and deploys (`AC-061`); TypeScript strict; layout and theme tokens; `id-ID`/`Asia/Jakarta` constants; error and 404 shells; CI typecheck and lint; preview environment isolated (`AC-073`) |
| **P2** | Auth + DB + media | Secure login and durable persistence on D1 + R2 | Schema migrated from empty and idempotent (`AC-062`); data survives redeploy (`AC-063`); login/session/CSRF/rate-limit verified (`AC-026`–`AC-032`, `AC-034`, `AC-067`); R2 upload pipeline with validation (`AC-064`, `AC-066`); batched writes (`AC-069`); password algorithm benchmarked and recorded |
| **P3** | CMS | Owner can manage all content | Products, articles, media library, settings, branding, SEO fields, and publishing workflow complete; `AC-033`, `AC-035`–`AC-048`, `AC-056`, `AC-057` pass |
| **P4** | Public pages | All public routes live from CMS content | Every page in §5.1 renders from the CMS; filters, inquiry CTAs, contact form; `AC-001`–`AC-025` pass |
| **P5** | SEO / performance / security hardening | Discoverable, fast, safe | Metadata, canonical, OG, sitemap, robots, JSON-LD, redirects, noindex rules; Lighthouse targets met; security headers enforced; `AC-036`, `AC-049`–`AC-055`, `AC-060` pass |
| **P6** | Launch | Production readiness | Domain and DNS on Cloudflare; R2 custom domain live; production secrets set via `wrangler secret`; backup verified and restore rehearsed (`AC-074`); monitoring on (`AC-078`); owner content entered; final manual a11y and responsive pass; cost model documented (`AC-077`); remaining `OQ`s closed |

#### 18.1.1 Phase 0 gate — implementation may not start until **all** of these are true

This is the gate referenced throughout `AGENTS.md`. It is deliberately hard: the platform changed once already, and a second pivot after code exists is far more expensive than a delay now.

| # | Gate item | Owner of the item | Status |
|---|---|---|---|
| G0-1 | **Framework signed off** — `OQ-013` answered in writing (Astro, or the React Router runner-up) | Owner | ☐ Not met |
| G0-2 | **Cloudflare account ready** and the person with admin access is named | Owner | ☐ Not met |
| G0-3 | **Zone / domain ready** — the domain is an active zone in that account | Owner | ☐ Not met |
| G0-4 | **D1 provisioned by the owner** — production and preview databases created, IDs recorded | Owner | ☐ Not met |
| G0-5 | **R2 provisioned by the owner** — bucket created, custom domain attached or hostname agreed | Owner | ☐ Not met |
| G0-6 | **Image strategy decided** — `OQ-014` resolved | Owner + implementer | ☐ Not met |
| G0-7 | **Password strategy decided** — `OQ-015` resolved after a benchmark on the real runtime | Implementer proposes, owner approves | ☐ Not met |
| G0-8 | **Workers plan tier decided** — `OQ-018` | Owner | ☐ Not met |
| G0-9 | **PRD v0.2 approved as a document** (status moves off "Draft — awaiting owner approval") | Owner | ☐ Not met |

The agent's obligation is simple: **if any row above is unchecked, do not scaffold, do not install a dependency, and do not provision a resource.** Ask, then wait.

Note on ownership: G0-2 through G0-5 are marked "by the owner" on purpose. Creating Cloudflare resources is an owner action with billing and access consequences; an agent must not create accounts, zones, databases or buckets on the owner's behalf.

### 18.2 Later-phase backlog (explicitly not in MVP)

- Scheduled publishing (publish at a future timestamp).
- Content revisions with diff and rollback.
- Multi-user roles (editor vs administrator).
- Privacy-friendly analytics integration.
- Product comparison view.
- Downloadable spec sheets or brochures (PDF) per product.
- Structured FAQ blocks with `FAQPage` JSON-LD (only with real, owner-authored FAQs).
- Multi-language support (requires a new PRD section and URL strategy).
- Automated image focal-point cropping.
- Email notification templates for inquiries.

### 18.3 Definition of done (per phase)

A phase is done only when: its requirements are implemented; the mapped `AC-###` are verified; typecheck and lint pass; no new unresolved security finding exists; affected documentation (`PRD.md`, `AGENTS.md`) is updated; and no secret, generated media, or database file is committed. For P1 onward, additionally: the change runs on the Workers runtime (not Node), no new binding or Cloudflare resource was added without approval, and any new dependency is recorded as Workers-compatible.

---

*End of PRD v0.2 — awaiting owner approval. Nothing in this document authorises implementation until the §18.1.1 gate is met.*
