# AGENTS.md — Garda Jaya CCTV

Operating instructions for any agent or developer working in this repository. Read this file first, then read `PRD.md`.

---

## 1. What this project is

Garda Jaya CCTV is a **business website plus an admin CMS** for an Indonesian company that sells, installs, and services CCTV systems for residential and commercial customers. The public site is in **Bahasa Indonesia** and exists to build trust and generate WhatsApp inquiries — it is not a shop. The CMS exists so the non-technical owner can change products, articles, branding, contact details, and SEO settings **without touching code or redeploying**. Every content type is data, not a hard-coded page.

Scope is defined in `PRD.md` §4. Non-goals there are binding: no checkout, no payments, no customer accounts, no inventory, no quotation engine, no multi-language, no mobile apps, no complex RBAC, no marketplace sync, no AI generation.

---

## 2. Sources of truth, in order

1. **`PRD.md`** — approved requirements. This is the authority on *what* to build and *under which constraints*.
2. **Official Cloudflare / framework documentation** — the authority on *how the platform actually behaves*. Cloudflare docs beat blog posts, forum answers, and your memory.
3. **`AGENTS.md`** (this file) — working rules for how to operate.

**When these conflict, stop and raise it.** Do not silently pick a winner. A PRD claim contradicted by current Cloudflare docs is a PRD defect to be reported, not worked around in code.

- **Read `PRD.md` before starting any work.** Do not infer requirements from chat history, from this file alone, or from what "seems reasonable."
- **Requirement-first development.** Before writing code, identify the `FR-###` you are implementing and the `AC-###` that proves it. If no requirement covers the work, stop and ask — do not invent scope.
- **Update the PRD when approved requirements change.** Add or revise rows, keep IDs unique, and extend the §15.1 coverage map in the same edit. A code change that contradicts the PRD is a bug, not a shortcut.
- If `PRD.md` and this file ever disagree, **the PRD wins** and this file must be corrected.
- `PRD.md` is **Draft v0.2 — awaiting owner approval**. Items marked *Proposed* or *Open* are **not** approved. Do not build irreversible commitments on top of them.
- **Never describe this PRD as approved.** It has never been approved, in any version.

---

## 3. Confirmed architecture — never change silently

| Layer | Direction | Status |
|---|---|---|
| Hosting / runtime | **Cloudflare Workers** | **Confirmed** (owner) |
| Database | **Cloudflare D1** (SQLite) | **Confirmed** (owner) |
| Media storage | **Cloudflare R2** | **Confirmed** (owner) |
| Language | TypeScript, `strict` | Confirmed |
| Data model | SQLite-compatible relational schema | Confirmed |
| Rendering | Server-rendered public HTML, SEO-first | Confirmed |
| **Framework** | **Not decided** — see `PRD.md` §12.7–§12.9 | **Proposed — awaiting owner sign-off** |
| ORM / query layer | Drizzle on D1, or plain prepared statements | Proposed |
| Password hashing | Algorithm **Open** (`OQ-015`) — PBKDF2-SHA256 leading | Open |
| Image optimisation | Mechanism **Open** (`OQ-014`) — no `sharp` in a Worker | Open |

**Framework status is the most important line in this table.** The PRD recommends Astro + `@astrojs/cloudflare`, with React Router v8 as runner-up, and records that recommendation as *Proposed*. **Do not assume Next.js, and do not assume Astro.** Do not scaffold, install, or configure a framework until `OQ-013` is answered and the Phase 0 gate (`PRD.md` §18.1.1) passes.

**Vercel is no longer the target.** Do not reintroduce Vercel-specific patterns: ISR, `@vercel/*` packages, Vercel Blob, `vercel.json`, "edge runtime" configuration, or the framing "the Vercel filesystem is ephemeral". The platform reason has changed — Workers has its own constraints (see §4). Any proposal that reintroduces a Vercel pattern requires an owner-approved PRD change, not a code review.

**Rule:** never silently change the Workers runtime, the D1 database, the R2 media store, the SQLite data-model direction, or the server-rendered SEO strategy. These are owner-level decisions. If you believe a change is needed, write the proposal into `PRD.md` (decision table + rationale + migration impact), then **wait for owner approval**. Swapping D1 for Postgres, or moving off Workers, is a PRD amendment — not a refactor.

Do not adopt a library, template, service, or provider that is marked *Proposed* or *Open* without explicit approval. Build against the documented **interfaces** (storage adapter, repository layer) so the choice stays reversible.

---

## 4. Hard constraints (non-negotiable)

1. **No database file in production.** There is no production filesystem. Workers' `node:fs` is a memory VFS: `/bundle` is read-only and `/tmp` is writable but **per-request and non-persistent**. A `.db` file in a deployment is a frozen, unwritable snapshot. Local D1 is emulated by Wrangler and reached through the **binding**, not a `file:` URL. Any production config resolving to `file:` is a release blocker.
2. **No writes to the filesystem for uploads.** Uploads stream to R2. The DB stores a storage key plus an absolute public URL on the R2 custom domain — never a local path, a relative path, a `/tmp` path, or an `r2.dev` URL in production content.
3. **No secrets in the repository.** Use `wrangler secret put` in production and `.dev.vars` locally (gitignored). Never put a secret in `wrangler.jsonc` `vars`, the CMS settings table, client bundles, or logs. Never commit `.dev.vars`, `.env*`, or `.wrangler/`.
4. **No generated media or database files committed.** No `.db`, `.sqlite`, dumps, uploads, build output, or derived image variants in Git.
5. **`logo/` is read-only reference material.** Do not edit, rename, re-encode, optimise in place, upscale, or auto-trace these files. Do not extract variants from the presentation mockup.
6. **No fabricated business facts.** Never invent certifications, testimonials, client logos, warranty periods, years of experience, project counts, awards, or partner claims. Contact details and product data come from the owner. If a value is unknown, omit the element — do not fill the gap.
7. **Runtime compatibility is a gate, not a hope.** No dependency that needs a real filesystem, a native addon, a long-lived process, or Node-only globals. Configuration comes from **bindings**, not `process.env`. Every new dependency is checked for Workers support before it is added (`NFR-058`).
8. **Respect the CPU budget.** Hashing, rendering, parsing and uploads run inside the per-invocation CPU budget documented in `PRD.md` §12.2 (the figure is not restated here). Nothing CPU-heavy goes on a request path without measurement (`NFR-042`).
9. **No interactive transactions.** D1 has no `BEGIN`…`COMMIT` across round trips. Multi-statement writes that must be atomic are one `batch()` call (`FR-185`).
10. **No Cloudflare resource is created by an agent.** Accounts, zones, D1 databases, R2 buckets and DNS records are owner actions (PRD §18.1.1).

---

## 5. Security expectations

- Single `Administrator` role for the MVP. Do not build RBAC, invitations, or permission matrices.
- Passwords stored as a **slow, salted, one-way hash** — never reversible, never logged. The **algorithm is Open** (`PRD.md` `OQ-015`): Argon2id/bcrypt parameters written for Node.js may blow the Workers CPU budget, so PBKDF2-SHA256 via Web Crypto is the leading candidate pending a benchmark on the real runtime. Do not hard-code an algorithm before that decision is recorded.
- **Every admin request re-validates the session server-side.** Middleware is a UX redirect only — it is not an authorization boundary. Each route handler and API route must independently enforce authentication.
- Sessions live server-side in D1 and are carried in a signed cookie. Signing uses **Web Crypto HMAC** (`crypto.subtle`), not a Node-only crypto module. Cookie: `HttpOnly`, `Secure` in production, `SameSite=Lax` or `Strict`, scoped path. No tokens in `localStorage`.
- **KV is not in scope.** Do not add a KV namespace for sessions or rate limiting without owner approval (`OQ-016`). D1 carries both.
- **Cloudflare Access (Zero Trust) in front of `/admin` is Open** (`OQ-015`, `D-22`). It may complement the app's login but never replaces it.
- CSRF protection on every state-changing request (origin/`Sec-Fetch-Site` check plus a per-session token).
- Login throttling (**product security decision proposed by this document**): 5 failed attempts per 15 minutes per IP+account, with a generic error that does not reveal whether an account exists. This is a design choice recorded in the PRD (`FR-086`, `AC-029`), not a Cloudflare limit.
- Validate and normalise all input at boundaries: route params, query strings, form bodies, uploads, settings.
- Sanitise rich text server-side on save **and** again on render. Never render unsanitised HTML.
- Uploads: verify content type against **magic bytes** (a browser-supplied `Content-Type` is untrusted), enforce a size limit, strip EXIF (including GPS), reject SVG from users, and generate random opaque object keys. If EXIF stripping cannot be verified for a format, reject that format rather than claiming it is stripped.
- Admin routes are `noindex`, disallowed in `robots.txt`, excluded from the sitemap, and never linked from public pages.
- Errors shown to users are generic; details go to server logs only.

---

## 6. SEO expectations

- Public pages are server-rendered: primary content must exist in HTML without JavaScript.
- One canonical URL per page, absolute, matching the served URL. Never canonicalise to a redirect or a 404.
- Per-page title and meta description come from CMS fields or documented defaults — unique per page.
- Structured data only when true: `Organization`/`LocalBusiness` requires confirmed business data (omit the node if a required field is missing); `Product` includes `offers` only when a real price exists; inquiry-only products emit no price.
- **Drafts and previews are never indexable** — enforce at render time plus `X-Robots-Tag: noindex` on preview responses.
- **A published slug change produces an HTTP 301** from the old URL to the new one, recorded in `slug_history`. Collapse redirect chains; block slug reuse while history is live.
- Every content image requires alt text; enforce it at publish time.
- **Never promise rankings, positions, traffic, or "SEO guarantees."** Do not put such claims in metadata, UI copy, or comments.

---

## 7. Code quality

- TypeScript `strict`. No `any` escapes to silence errors — model the type or validate at the boundary.
- Validate at boundaries and let the interior work with trusted types.
- Small, focused modules with one clear responsibility. Prefer composition over inheritance.
- No dead code, no commented-out blocks, no unused exports, no "temporary" flags left behind.
- Semantic, accessible HTML: landmarks, one `h1` per page, logical heading order, labelled form controls, visible focus, keyboard operability. Target WCAG 2.1 AA.
- All user-facing strings centralised (Bahasa Indonesia for public and admin UI); `id-ID` locale and `Asia/Jakarta` timezone defined once as constants.
- Format money as integer rupiah. **Never render an empty price or `Rp 0`** — branch on `price_display` (`fixed` / `from` / `range` / `inquiry`) and fall back to the inquiry label.

---

## 8. Third-party licence gate

Before adding **any** dependency, template, font, icon set, or image:

1. Read the actual `LICENSE` file — not the README, not the marketing page, not a badge.
2. Confirm **commercial use** is permitted. A public repository does **not** imply free commercial use.
3. Check that no copyleft obligation would force this project's source to be disclosed.
4. Record the decision (package, version, licence, reason) in the PRD decision table or PR review.
5. Reject anything with an unclear licence, a "free tier" that gates essential features, or a paid component dependency behind a free shell.

The admin template is **evaluation-only** right now (PRD §7.10). No template has been selected, installed, or committed. Complete the gate criteria — licence, framework compatibility, maintenance activity, responsiveness, accessibility, branding ease, dependency posture, and the rule that the public site must not look like a dashboard — before adopting one.

---

## 9. Testing & verification — **planned, no tooling exists yet**

There is currently no test runner, linter, or CI configuration in this repository. The following is the intended verification strategy once tooling is introduced; **do not describe these as existing commands.**

| Layer | Planned coverage |
|---|---|
| Type checking | TypeScript strict, whole project, in CI |
| Lint | Configured ruleset, no warnings tolerated in CI |
| Unit | Slug generation and normalisation, redirect resolution and chain collapsing, price-display branching, settings validation, WhatsApp number normalisation |
| Integration | Publish gate validation, slug-history writes, **`batch()` atomicity with fault injection**, D1 migrations from an empty database (and idempotent re-run), media adapter contract, magic-byte/size rejection |
| End-to-end | Login and logout, admin route protection when logged out (`curl`, no UI), draft → preview → publish, unpublish, slug change → 301, product creation with all four price modes, image upload → R2 URL survives redeploy |
| Platform | `wrangler dev` run, `wrangler deploy --dry-run`, preview-environment isolation check, secret scan of the client bundle |
| Manual | Responsive pass at 320/360/768/1024/1280 px; keyboard-only navigation; screen-reader spot check; contrast verification |

Minimum bar for any change: it type-checks, it lints clean, it runs on the Workers runtime (not Node), and the `AC-###` it claims to satisfy is demonstrably verified.

---

## 10. Environment, data & deployment

- **Host:** Cloudflare Workers. Local development uses `wrangler dev`, which runs the Worker in the Workers runtime — not Node.
- **Environments:** local, preview/staging, production. Preview uses its **own** D1 database and R2 bucket; production data must never be reachable from preview (`FR-195`).
- **Region:** Workers run at the edge; D1 has a primary location. Placement and latency must be **measured**, not assumed (`OQ-018`).
- **Timezone:** `Asia/Jakarta` (UTC+7). Store timestamps in UTC; render in Jakarta time.
- **Locale:** `id-ID`. Public copy in Bahasa Indonesia; admin UI in Bahasa Indonesia.
- **Production persistence:** D1 only. The Worker filesystem is never a data store.
- **Migrations:** version-controlled `.sql` files in `migrations/`, applied with `wrangler d1 migrations apply` as a deliberate step. **Never auto-migrate on request in production.** Never hand-edit a migration that has been applied anywhere.
- **Atomic writes:** one `batch()` per multi-statement operation. No `BEGIN` held across awaits (`FR-185`).
- **Cache invalidation:** publishing or changing settings must **purge** the affected cached URLs so content appears without a redeploy. There is no ISR — see `PRD.md` §12.4.
- **Secrets:** `wrangler secret put` (production), `.dev.vars` (local, gitignored). Never in `wrangler.jsonc` `vars`.

### 10.1 D1 rules

- Migrations **will live in** `migrations/` under version control once the implementation phase begins, and will be applied **only** with `wrangler d1 migrations apply` — local, then preview, then production, deliberately. (There are no migrations yet; §13.2.1 describes the mechanism, not a present file tree.)
- **Never hand-edit a migration that has been applied anywhere.** Add a new one. (There is nothing to edit yet.)
- **Never `DROP`** a table, column, or index without explicit approval and a tested backup.
- **No interactive transactions.** Multi-statement writes that must be atomic are a single `batch()`. Do not `BEGIN`, await, then `COMMIT`.
- Access D1 through the binding (`env.DB`). No connection string, no credentials, no `file:` URL.
- Keep queries indexed; a public route must not perform an unbounded table scan.

### 10.2 R2 rules

- Uploads go to R2 only. Never write uploads to the runtime filesystem — `/tmp` is per-request and vanishes.
- Validate **MIME + magic bytes + size** server-side before storage. A browser-supplied `Content-Type` is untrusted.
- Keys are random and opaque. Never derive a key from a filename or from upload order.
- Store both the storage key and the absolute public URL in D1; the URL must be on the R2 **custom domain**, never `r2.dev`.
- Delete the R2 object **before** the D1 row, and rely on reconciliation to catch orphans. The two systems cannot share a transaction.
- Large uploads may use a presigned PUT — issued only to an authenticated, CSRF-checked request, with a short expiry and server-side verification afterwards.

---

## 11. Working etiquette

- **Ask before** adding a dependency, adopting a service, changing the schema, altering the auth model, or introducing a new route pattern.
- Keep changes scoped to the requirement being implemented. Unrelated refactors belong in a separate change.
- Commits are small and descriptive; reference the requirement ID(s) they implement (e.g. `feat(products): inquiry price fallback (FR-046, AC-018)`).
- Never commit secrets, generated media, database files, or build output.
- When a decision is genuinely ambiguous, prefer the conservative option that is easiest to reverse, and record the ambiguity as an open question rather than silently choosing.
- Update `PRD.md` and this file in the same change when behaviour or constraints shift.

---

## 12. Do not

- Do not build checkout, cart, payments, orders, or any e-commerce flow.
- Do not invent testimonials, certifications, client logos, awards, warranties, years of experience, or project counts.
- Do not let admin UI styling, components, or scripts leak into public pages.
- Do not run silent or automatic schema migrations, and do not hand-edit an applied migration.
- Do not edit, rename, upscale, or trace the files in `logo/`.
- Do not write to the filesystem for uploads or data.
- Do not commit a `.db` file, a dump, or use a local database file in production.
- Do not store secrets in the CMS, the repository, `wrangler.jsonc` `vars`, or client bundles.
- Do not add a second language, a language switcher, or multi-locale routing.
- Do not expose admin routes in navigation, the sitemap, or `robots.txt` allow rules.
- Do not promise SEO rankings or traffic outcomes.
- Do not reference build, lint, test, or dev commands as if they exist today.
- Do not reintroduce Vercel patterns or describe the platform as Vercel (§3).
- Do not scaffold a framework, create a Cloudflare resource, or add a KV namespace / new binding before approval (§4, §14).
- Do not use `process.env` for configuration, hold a transaction open, or add `sharp` (§4, §10).

---

## 13. Current repository state (re-verified 2026-10-26)

What exists — **this is the entire repository**:

```
AGENTS.md                  ← this file
PRD.md                     ← product requirements, source of truth
logo/
  Garda Jaya CCTV Logo Presentation.png              (presentation sheet — NOT a web asset)
  Garda-Jaya-CCTV-Main-Logo-Monochrome-Navy.png      (single-colour navy, light backgrounds/print)
  Garda-Jaya-CCTV-Main-Logo-Only.png                 (icon mark — favicon/app icon source)
  Garda-Jaya-CCTV-Main-Logo.png                      (primary lockup)
```

Git state (verified, not assumed): branch `main` with **exactly one commit** (`342f0c1`, adding the logo files). Tracked: `AGENTS.md`, `PRD.md`, three logo PNGs. Untracked but now gitignored: `logo/Garda Jaya CCTV Logo Presentation.png` (the presentation sheet is intentionally excluded from version control; `.gitignore` appeared after the audit from outside this task).

What does **not** exist yet — do not assume or reference it:

- No `package.json`, no lockfile, no `node_modules`.
- **No `wrangler.jsonc` / `wrangler.toml`**, no `migrations/`, no `.dev.vars`.
- No `tsconfig.json`, framework config, linter config, or test config.
- No `src/`, `app/`, `pages/`, `public/`, `components/`, or API routes.
- No scripts, no CI pipeline, no build or dev server.
- No Cloudflare resource of any kind — no account setup, no D1 database, no R2 bucket, no zone, no DNS.

Consequences: there are **no commands to run** and **no conventions to inherit**. Setup (proper `.gitignore` covering `.dev.vars`/`.wrangler/`/build output, project scaffold, tooling) is Phase 1 work in `PRD.md` §18 — not something to assume has happened. Known asset gaps (no vector master, no delivered white-on-navy variant, non-square 300×250 icon) are documented in `PRD.md` §10.2 and must not be worked around by fabricating assets.

---

## 14. Phase gate — do not build yet

Implementation is **blocked** until every item in `PRD.md` §18.1.1 is met: framework signed off (`OQ-013`), Cloudflare account and zone ready, D1 and R2 provisioned **by the owner**, image strategy decided (`OQ-014`), password strategy benchmarked (`OQ-015`), plan tier decided (`OQ-018`), and PRD v0.2 approved as a document.

If any item is unmet: **ask, then wait.** Do not scaffold, install a dependency, create a resource, or "prepare" a project that presumes an unmade decision.

---

*This file is durable guidance, not a copy of the PRD. When requirements change, update `PRD.md` first.*

*Last revised: 2026-10-26 — platform switched to Cloudflare Workers + D1 + R2; framework reopened and left Proposed (`PRD.md` §12).*
