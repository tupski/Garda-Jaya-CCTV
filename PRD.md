# PRD — Garda Jaya CCTV Website & Admin CMS

**Product:** Garda Jaya CCTV — business website + admin CMS
**Document version:** v0.1
**Date:** 2026-10-26
**Status:** **Draft v0.1 — awaiting owner approval**
**Owner:** Garda Jaya CCTV (business owner)
**Prepared by:** Documentation phase — no implementation performed

---

## 1. Document purpose, status, audience, change log, ID conventions

### 1.1 Purpose

This document is the single source of truth for *what* will be built, *why*, and *under which constraints*. It exists so any agent or developer can implement the product without re-deriving decisions from chat history, and so the business owner can approve or reject each unresolved item explicitly.

This PRD is **documentation only**. No application code, dependency, schema, or configuration exists yet. Nothing here authorises installing packages, creating a database, or selecting a vendor.

### 1.2 Status

Draft v0.1. Every item marked **Proposed** or **Open** (see the §13.4 decision table and §17) requires the owner's explicit sign-off before implementation. Items marked **Confirmed** reflect decisions already given by the owner.

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
| v0.1 | 2026-10-26 | Documentation phase | Initial draft: scope, requirement registers, persistence decisions, risks, open questions, phased delivery |

### 1.5 How to read requirement IDs

Every requirement has a stable, never-reused ID. IDs are referenced by acceptance criteria (§15) and by agents when planning work.

| Prefix | Meaning | Range reserved |
|---|---|---|
| `FR-###` | Functional requirement | `FR-001`–`FR-179` (blocked by domain) |
| `NFR-###` | Non-functional requirement | `NFR-001`–`NFR-040` |
| `AC-###` | Acceptance criterion | `AC-001`–`AC-060` |
| `DEP-###` | Dependency | `DEP-001`–`DEP-010` |
| `RISK-###` | Risk | `RISK-001`–`RISK-010` |
| `OQ-###` | Open question requiring owner decision | `OQ-001`–`OQ-012` |

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
| `FR-160`–`FR-179` | Publishing workflow, status model, redirects |

Rules:

1. A requirement is never deleted. If dropped, its status becomes *retired* and the ID is never reused. Unused numbers inside a block remain reserved and must not be reassigned to another domain.
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
| O5 | Stay maintainable and low-cost for a small business | Vercel plus free-tier-compatible data services, documented swap points |

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

### 2.6 Current repository state (verified 2026-10-26)

The repository was audited with file tools before this document was written.

- **Present:** `logo/` containing exactly 4 PNG files (§10.1).
- **Absent:** no `AGENTS.md`, no `README.md`, no `package.json`, no lockfile, no `next.config.*`, no `tsconfig.json`, no `.env*`, no `.gitignore`, no `src/`, no `app/`, no `public/`, no scripts, no tests, no CI configuration.
- **Git:** at the start of this documentation phase the folder was **not** a Git repository (`git status` returned "not a git repository"). A `.git` directory was created afterwards, during the session. It currently has **zero commits** (branch `main`, no history), and `.gitignore` does not exist. `git status` reports only `AGENTS.md`, `PRD.md`, and `logo/` as untracked.

Consequences binding all later work:

1. There are **no existing conventions to follow** and **no conflicting files**. No file was overwritten.
2. No build, lint, test, or dev command exists. Any command reference in planning documents is *planned*, not current.
3. The repository has no commits and no `.gitignore`. Creating `.gitignore` and making the initial commit are Phase 1 items, not assumptions.
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
| Persistence | SQLite-compatible relational model; production persistence via an approved managed service; media via object storage |
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

Uploads are validated, transformed, stored in external object storage, and referenced by CDN URL. No upload is ever written to the deployment filesystem as the source of truth (§13.3). Deleting referenced media warns and requires confirmation.

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
| `FR-081` | Passwords stored with Argon2id (preferred) or bcrypt cost ≥ 12; never plaintext, never logged | MUST | P2 | `AC-027` |
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
| `FR-101` | Storage access goes through a provider-agnostic adapter interface (put/delete/getPublicUrl/replace) | MUST | P2 | `AC-038` |
| `FR-102` | Object keys are random and opaque; never derived from original filename or sequential | MUST | P2 | `AC-039` |
| `FR-103` | Content-type validated against magic bytes; mismatches rejected | MUST | P2 | `AC-037` |
| `FR-104` | Per-file size limit enforced server-side with a clear Indonesian error | MUST | P2 | `AC-037` |
| `FR-105` | EXIF (including GPS) stripped on upload | MUST | P2 | `AC-040` |
| `FR-106` | Responsive width variants generated and referenced via `srcset` with explicit dimensions | SHOULD | P2 | `AC-040` |
| `FR-107` | Public absolute CDN URL stored in the database; never a local or deployment-internal path | MUST | P2 | `AC-038` |
| `FR-108` | Media library grid with search, type filter, pagination, and metadata display | MUST | P3 | `AC-041` |
| `FR-109` | Deletion checks references, warns, requires confirmation, then removes the object and row | MUST | P3 | `AC-042` |
| `FR-110` | Alt text required for content images; enforced at publish | MUST | P3 | `AC-041` |
| `FR-111` | Media replacement preserves the media record and existing references | SHOULD | P3 | `AC-043` |
| `FR-112` | Failed uploads leave no orphan database row and no partial object | SHOULD | P2 | `AC-042` |
| `FR-113` | Delivery uses optimised formats with fallbacks and lazy loading below the fold | SHOULD | P5 | `AC-040` |
| `FR-114` | Accepted formats restricted (jpeg/png/webp/avif); SVG rejected for user uploads | MUST | P2 | `AC-037` |
| `FR-115` | Oversized images downscaled on upload to a maximum dimension | SHOULD | P2 | `AC-040` |
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
| `FR-125` | Settings and content saves trigger revalidation of affected public paths without a redeploy | MUST | P3 | `AC-047` |
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
3. Publish runs validation (required fields, slug uniqueness, price consistency, image alt text), then flips status and revalidates affected public paths (`FR-162`, `FR-179`).

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
| `FR-179` | Publish/unpublish revalidates all affected public paths | MUST | P3 | `AC-047` |

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
| Presentation sheet is opaque and 1.18 MB | Unsuitable as a web asset | Never ship it; reference only |

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
| Password storage | Argon2id (preferred) or bcrypt cost ≥ 12; never reversible, never logged (`FR-081`) |
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

---

## 12. Technology & deployment constraints

### 12.1 Mandatory direction

| Layer | Decision | Status |
|---|---|---|
| Runtime | Node.js + TypeScript | **Confirmed** |
| Framework | Next.js (App Router) + React + TypeScript | **Proposed — awaiting owner sign-off** |
| Hosting | Vercel | **Confirmed** |
| Data model | SQLite-compatible relational (SQL, single-file heritage) | **Confirmed** |
| Rendering | Public pages server-rendered / statically generated; SEO-first | **Confirmed** |

### 12.2 Framework justification (Next.js App Router)

| Reason | Detail |
|---|---|
| Vercel alignment | First-class support, zero-config deploys, preview deployments, image optimisation |
| SEO capability | SSR/SSG/ISR produce complete HTML per page; metadata API for titles, canonical, OG |
| CMS API in the same app | Route Handlers provide the admin API without a second service |
| Routing | File-based routes map cleanly to the Indonesian slug sitemap (§5.1) |
| Ecosystem & longevity | Largest React meta-framework ecosystem; long-term maintenance and hiring pool |
| Constraint fit | Works with SQLite-compatible HTTP drivers (libSQL) from serverless functions |

Alternatives considered and not chosen: Astro (excellent for content, but the admin API and interactive editing push complexity into a second service), SvelteKit/Remix (viable, smaller ecosystem for long-term maintenance by a small business), WordPress (rejected: the requirement is a bespoke brand and a controlled stack, and the SQLite direction would be lost).

**This choice is documented, not final.** The owner approved "Node.js + Vercel + SQLite direction" and delegated framework selection to documentation. Next.js is therefore the working choice while remaining listed as *Proposed — awaiting owner sign-off* in §13.4. Swapping frameworks later requires a PRD revision.

### 12.3 Vercel constraints that shape the design

| Constraint | Consequence |
|---|---|
| Serverless/edge functions are stateless and short-lived | No in-memory cache as source of truth; sessions and data live externally |
| Deployment filesystem is read-only at runtime; `/tmp` is ephemeral | **No SQLite file writes; no upload writes** (§13.2, §13.3) |
| Each deployment is a new immutable bundle | A `.db` inside the bundle is a frozen snapshot; writes are lost |
| Function duration and payload limits | Uploads go to object storage (direct or pre-signed) rather than through long-running functions where avoidable |
| Cold starts | Keep DB connections per-invocation; use HTTP-based drivers |
| Build-time vs runtime env | Secrets must be runtime env vars, never build-time inlined into the client |

Region: choose the Vercel region closest to Indonesian users and to the database primary (e.g. Singapore) — **to be verified** (`OQ-007`).

---

## 13. Database & media persistence decisions

### 13.1 Four separate concepts (do not conflate)

1. **SQLite as the data model.** The schema is relational and SQL-based, designed with single-file SQLite heritage (no Postgres-only types, no stored procedures, no `jsonb` operators, no arrays). This is a *modelling* decision and it is **Confirmed**.
2. **Local SQLite file for development only.** Developers may run against `file:./local.db`. That file is gitignored, never seeded with real customer data, and never referenced by production configuration.
3. **Production persistence on Vercel.** Vercel's runtime filesystem is read-only and ephemeral. A `.db` file bundled into a deployment cannot accept writes and cannot persist across deployments or invocations. Therefore **a local `.db` file is prohibited in production.** This is a hard constraint, not a preference.
4. **Proposed production option: Turso (libSQL).** A managed, SQLite-compatible service speaking an HTTP/replica protocol, usable from serverless functions. **Not finalised.** It requires owner approval for vendor dependency, free-tier limits, and cost at scale.

### 13.2 Production database options

| Option | Fit | Trade-off | Status |
|---|---|---|---|
| **Turso (libSQL)** — recommended | SQLite-compatible, HTTP driver, works from Vercel functions, local parity via libSQL file | Vendor dependency; free-tier limits and pricing **to be verified**; replication/consistency semantics must be understood | **Proposed** |
| Self-hosted libSQL server | Keeps SQLite semantics, no SaaS vendor | Requires a host, backups, patching, monitoring — operational burden for a small business | Alternative |
| Serverless Postgres (Neon / Vercel Postgres) | Excellent serverless fit, mature backups | **Deviation**: replaces the approved SQLite direction; requires amending this PRD | Alternative — requires approval |
| SQLite file on Vercel | — | **Prohibited**: read-only/ephemeral filesystem, data loss | Rejected |
| SQLite on a persistent VM/volume | Technically possible | Contradicts the Vercel-mandatory deployment; new operational burden | Rejected |

> Any provider recommendation must be **re-verified against current official documentation at implementation time**. Do not assert specific current pricing, row limits, storage limits, or replication behaviour as fact in code comments, marketing, or planning — describe them as "to be verified".

### 13.3 Media persistence

Uploads written to the Vercel runtime filesystem are lost or fail (read-only). Media must live in object storage and be referenced by public CDN URL.

| Candidate | Notes | Status |
|---|---|---|
| **Vercel Blob** | Native to the platform, simple token setup, CDN delivery | Pending approval |
| **Cloudinary** | Strong transformation pipeline; free tier **to be verified** | Pending approval |
| **S3-compatible (e.g. Cloudflare R2)** | Low egress cost, portable, standard API | Pending approval |
| **Supabase Storage** | S3-like with policies; adds a second vendor surface | Pending approval |

Requirements for whichever provider is approved: public CDN URL stored in the database (`FR-107`), random unguessable keys (`FR-102`), content-type plus magic-byte validation (`FR-103`), per-file size limit (`FR-104`), EXIF stripping (`FR-105`), responsive variants (`FR-106`), deletion cleanup (`FR-109`), and a swap-in adapter interface (`FR-101`).

Adapter interface (shape only — not an implementation):

```
putObject({ key, body, contentType }) -> { url, key }
deleteObject({ key }) -> void
getPublicUrl({ key, variant? }) -> string
replaceObject({ key, body, contentType }) -> { url, key }
```

Business logic depends on the interface, never on a provider SDK directly. Swapping providers must be one adapter implementation plus a migration of stored URLs.

### 13.4 Decision table

| ID | Area | Leading option | Status | Owner action |
|---|---|---|---|---|
| D-01 | Runtime | Node.js + TypeScript | **Confirmed** | — |
| D-02 | Framework | Next.js App Router + React + TypeScript | **Proposed** | Sign off or request alternative |
| D-03 | Hosting | Vercel | **Confirmed** | — |
| D-04 | Data model | SQLite-compatible relational schema | **Confirmed** | — |
| D-05 | Local dev DB | `file:./local.db`, gitignored | **Confirmed** (constraint) | — |
| D-06 | Production DB | Turso (libSQL) | **Proposed** | Approve vendor + accept free-tier limits |
| D-07 | ORM | Drizzle ORM + `@libsql/client` (Prisma as alternative) | **Proposed** | Sign off |
| D-08 | Media storage | Vercel Blob / Cloudinary / R2 / Supabase Storage | **Open** | Choose one |
| D-09 | Admin template | None selected — evaluation only (§7.10) | **Open** | Approve shortlist, then one template |
| D-10 | Auth library | Auth.js or vetted session library | **Proposed** | Sign off |
| D-11 | Rich text editor | Tiptap / Lexical / Markdown+preview | **Open** | Approve after trial |
| D-12 | Rich text storage | Sanitised HTML vs structured JSON | **Open** | Decide with editor choice |
| D-13 | Vercel region | Closest to users + DB (e.g. Singapore) | **Open** | Confirm after D-06 |
| D-14 | Domain & DNS | Owner-provided domain | **Open** | Provide domain |
| D-15 | Backups | Provider snapshots + scheduled logical dump | **Proposed** | Approve retention |
| D-16 | Analytics | None in MVP (optional privacy-friendly later) | **Open** | Confirm deferral |

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

- Migrations are versioned SQL files generated by the ORM and applied as an explicit step — **never** automatically on request in production (`RISK-008`).
- Seed data contains **no fabricated business facts** and no fake products or articles. Development seeds may use clearly-marked dummy records, gated to development only.
- The administrator account is created from environment configuration or a one-time setup flow, never from a committed seed with a default password.
- Development and production run the **same schema**; only the driver/URL differs (`NFR-017`). SQLite dialect differences between a local libSQL file and remote libSQL must be exercised in tests.
- Data must survive redeployments and framework upgrades (`NFR-016`, `AC-058`).

---

## 14. Non-functional requirements

| ID | Requirement | Target / condition | Priority | Phase | Verification |
|---|---|---|---|---|---|
| `NFR-001` | Core Web Vitals on public pages | LCP ≤ 2.5 s, CLS < 0.1, INP < 200 ms at p75; Lighthouse mobile (4× CPU throttle, Slow 4G), production build on a Vercel preview URL, cold and warm cache | MUST | P5 | `AC-060` |
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
The `users` table contains only a password hash, never plaintext. The hash is Argon2id or bcrypt cost ≥ 12. Correct password succeeds, wrong password fails, and no password appears in logs or error responses.

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
Given a product image referenced by a published product, replacing the file shows the new image after revalidation, keeps the media record ID unchanged, and breaks no other page referencing that media.

**`AC-044`** (FR-120, FR-124, FR-136)
Uploading a new primary logo and saving settings updates the public header on the next request; the old asset is not auto-deleted; an empty logo slot renders a text fallback rather than a broken image. Favicon and app-icon slots accept uploads and derived sizes are served.

**`AC-045`** (FR-121, FR-130, FR-131, FR-132)
Updating phone, email, address, operating hours, social links, and maps URL updates header, footer, `/kontak`, and the WhatsApp CTA without a redeploy. A social link left empty renders nothing.

**`AC-046`** (FR-123, FR-126, FR-133, FR-134, FR-137)
Brand tokens accept only validated formats; invalid values are rejected with a message and never persisted; a preview shows the effect before saving; contrast warnings appear for combinations failing AA. Resetting a settings group restores documented defaults. Typography options are constrained to approved choices.

**`AC-047`** (FR-125, FR-179)
Given a settings or content change, saving revalidates the affected public paths so the next request shows fresh content, while unaffected cached pages remain cached.

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
Given a production build measured with Lighthouse mobile (4× CPU throttle, Slow 4G) on a Vercel preview URL: LCP ≤ 2.5 s, CLS < 0.1, INP < 200 ms, accessibility score ≥ 95, no horizontal scroll at 320/360/768/1024/1280 px. Checklist: skip link, landmarks, one `h1`, visible focus, labelled inputs, 44 px touch targets, `srcset` with explicit dimensions, lazy loading below the fold.

---

## 16. Dependencies & risks

### 16.1 Dependencies

| ID | Dependency | Impact if unavailable | Mitigation |
|---|---|---|---|
| `DEP-001` | Owner-supplied business facts (address, phone, hours, service area, real product data) | Cannot publish trust-building content or `LocalBusiness` JSON-LD; site launches with thin content | Collect via a structured content checklist before P4; publish only confirmed data; omit rather than invent |
| `DEP-002` | Vercel account, project, and environment configuration | No deployment target | Provision in P1; document required env vars in `.env.example` once it exists |
| `DEP-003` | Production database service (Turso or approved alternative) | No production persistence; app cannot launch | Resolve `OQ-002` before P2; adapter pattern keeps the driver swappable |
| `DEP-004` | Object storage provider + credentials | No media uploads; no product or article images | Resolve `OQ-003` before P2; build against the adapter with a development-only driver |
| `DEP-005` | Domain name and DNS control | No canonical production URLs; SEO work is provisional | Resolve `OQ-007`; use the Vercel domain until then |
| `DEP-006` | Admin template licence verification | Admin UI cannot ship on an unverified licence | Complete §7.10 gate in P3; fall back to a custom minimal shell |
| `DEP-007` | Vector logo master and reversed/white variant from the designer | Brand cannot be applied to dark surfaces; favicons limited | Track as a brand task; use documented interim treatment (§10.2) |
| `DEP-008` | Rich text editor selection and storage format | Article and product bodies cannot be authored | Resolve `OQ-005` in P3; start with a minimal editor if delayed |
| `DEP-009` | Node.js LTS runtime availability on Vercel | Build failure or forced runtime change | Pin an LTS version; re-verify support at implementation time |
| `DEP-010` | Owner availability for content entry and review | Launch slips; placeholder content risk | Schedule content entry in P6 with a written checklist |

### 16.2 Risks

| ID | Risk | Impact | Mitigation |
|---|---|---|---|
| `RISK-001` | **Turso vendor lock-in** | Migration cost if the vendor changes terms or pricing | Adapter/repository layer; SQLite-compatible schema; documented export path; no vendor-specific SQL in business logic |
| `RISK-002` | **Free-tier limits** on database or storage (rows, bandwidth, storage, requests) | Site degrades or stops at traffic spikes | Monitor usage; document limits as "to be verified"; owner approves a paid tier before launch if needed |
| `RISK-003` | Local SQLite file accidentally used in production | Silent data loss on every deploy | Hard rule in `AGENTS.md`; startup check rejecting `file:` URLs in production; `AC-058` |
| `RISK-004` | Uploads written to the runtime filesystem | Images vanish after deploy; broken pages | Storage adapter only; no `fs` writes to app directories; `AC-059` |
| `RISK-005` | **Admin template licence risk** — public repo mistaken for commercial-use permission | Legal exposure and forced rework | Read the `LICENSE` file for every candidate; record findings; no template without a recorded licence decision |
| `RISK-006` | **Fabricated trust claims** (testimonials, certifications, years, counts) | Reputational and legal harm; misleading advertising | Editorial rule (§6.7); review before launch; no prefilled claims in templates or seeds |
| `RISK-007` | **Unknown contact details** at build time | Placeholder phone/address published | Treat as blocking for launch; hide unset fields; never invent data |
| `RISK-008` | **Silent schema migration** on deploy | Data corruption or downtime | Explicit migration step; no auto-migrate on request; tested against a restored copy |
| `RISK-009` | Admin UI leaking into public pages (shared CSS/JS) | Public site looks like a dashboard; performance and brand damage | Separate layouts and style boundaries; audit public bundles for admin code |
| `RISK-010` | **Scope creep toward e-commerce** (cart, checkout, stock, quotes) | Timeline blowout; architectural churn | Non-goals are explicit (§4.2); any change requires a PRD revision and owner approval |

---

## 17. Open questions requiring owner approval

| ID | Question | Why it matters | Default assumption if unanswered |
|---|---|---|---|
| `OQ-001` | Approve **Next.js (App Router)** as the framework? | Locks implementation approach and hosting behaviour | Proceed with Next.js as documented, treated as provisional until confirmed |
| `OQ-002` | Approve **Turso (libSQL)** as the production database? | Vendor dependency, free-tier limits, cost at scale | Build against the libSQL adapter; do not deploy production data until approved |
| `OQ-003` | Which **media storage provider** (Vercel Blob / Cloudinary / R2 / Supabase)? | Upload flow, cost, CDN behaviour | Implement the adapter; use a development-only local driver, never in production |
| `OQ-004` | Approve **Auth.js** or a vetted session library, vs. a hand-rolled session layer? | Security risk surface and maintenance | Prefer the vetted library; if unanswered, a hand-rolled implementation must satisfy §11 in full |
| `OQ-005` | Store rich text as **sanitised HTML** or **structured JSON**? | Editor choice, rendering, migration flexibility | Sanitised HTML with a strict allowlist, re-sanitised at render |
| `OQ-006` | Which **admin template** after licence verification? | Legal exposure and design consistency | Build a minimal custom admin shell with no third-party template |
| `OQ-007` | Preferred **Vercel region** (e.g. Singapore) and **domain name**? | Latency, canonical URLs, SEO | Singapore region; use the Vercel preview domain until a domain is provided |
| `OQ-008` | Approve **Kebijakan Privasi** wording; is counsel review required? | Legal accuracy and UU PDP context | Publish a plain-language draft marked for owner review; make no compliance claims |
| `OQ-009` | Is the **admin UI language** Bahasa Indonesia? | Usability for the non-technical owner | Bahasa Indonesia for all admin labels |
| `OQ-010` | Enable the **contact form**, or WhatsApp-only? | Scope, spam handling, data storage, privacy obligations | Contact form enabled and stored; disable if the owner prefers WhatsApp-only |
| `OQ-011` | Approve **backup retention** and the RPO/RTO targets in `NFR-008`? | Data-loss exposure | Daily logical backup, 7-day retention; RPO 24 h / RTO 4 h |
| `OQ-012` | Confirm **no analytics** in MVP, or approve a privacy-friendly option later? | Scope and privacy policy content | No analytics in MVP |

---

## 18. Phased delivery

### 18.1 Phases

| Phase | Name | Goal | Exit criteria |
|---|---|---|---|
| **P0** | Documentation | Agree scope, stack, persistence, and open questions | This PRD and `AGENTS.md` approved; `OQ-001`–`OQ-005` answered; repository initialised with `.gitignore` |
| **P1** | Foundation | Runnable skeleton with the correct rendering strategy | App builds and runs; TypeScript strict; layout and theme tokens; `id-ID`/`Asia/Jakarta` constants; error and 404 shells; CI typecheck and lint |
| **P2** | Auth + DB + media | Secure login and durable persistence | Schema migrated; admin provisioning works; login/session/CSRF/rate-limit verified (`AC-026`–`AC-032`, `AC-034`); upload adapter works against the approved provider; `AC-058`, `AC-059` pass |
| **P3** | CMS | Owner can manage all content | Products, articles, media library, settings, branding, SEO fields, and publishing workflow complete; `AC-033`, `AC-035`–`AC-048`, `AC-056`, `AC-057` pass |
| **P4** | Public pages | All public routes live from CMS content | Every page in §5.1 renders from the CMS; filters, inquiry CTAs, contact form; `AC-001`–`AC-025` pass |
| **P5** | SEO / performance / security hardening | Discoverable, fast, safe | Metadata, canonical, OG, sitemap, robots, JSON-LD, redirects, noindex rules; Lighthouse targets met; security headers enforced; `AC-036`, `AC-049`–`AC-055`, `AC-060` pass |
| **P6** | Launch | Production readiness | Domain and DNS; production env vars; backup verified and restore rehearsed; monitoring; owner content entered; final manual a11y and responsive pass; `OQ-006`–`OQ-012` closed |

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

A phase is done only when: its requirements are implemented; the mapped `AC-###` are verified; typecheck and lint pass; no new unresolved security finding exists; affected documentation (`PRD.md`, `AGENTS.md`) is updated; and no secret, generated media, or database file is committed.

---

*End of PRD v0.1 — awaiting owner approval.*
