# AGENTS.md — Garda Jaya CCTV

Operating instructions for any agent or developer working in this repository. Read this file first, then read `PRD.md`.

---

## 1. What this project is

Garda Jaya CCTV is a **business website plus an admin CMS** for an Indonesian company that sells, installs, and services CCTV systems for residential and commercial customers. The public site is in **Bahasa Indonesia** and exists to build trust and generate WhatsApp inquiries — it is not a shop. The CMS exists so the non-technical owner can change products, articles, branding, contact details, and SEO settings **without touching code or redeploying**. Every content type is data, not a hard-coded page.

Scope is defined in `PRD.md` §4. Non-goals there are binding: no checkout, no payments, no customer accounts, no inventory, no quotation engine, no multi-language, no mobile apps, no complex RBAC, no marketplace sync, no AI generation.

---

## 2. Documentation is the source of truth

- **Read `PRD.md` before starting any work.** Do not infer requirements from chat history, from this file alone, or from what "seems reasonable."
- **Requirement-first development.** Before writing code, identify the `FR-###` you are implementing and the `AC-###` that proves it. If no requirement covers the work, stop and ask — do not invent scope.
- **Update the PRD when approved requirements change.** Add or revise rows, keep IDs unique, and extend the §15.1 coverage map in the same edit. A code change that contradicts the PRD is a bug, not a shortcut.
- If `PRD.md` and this file ever disagree, **the PRD wins** and this file must be corrected.
- `PRD.md` is **Draft v0.1 — awaiting owner approval**. Items marked *Proposed* or *Open* are **not** approved. Do not build irreversible commitments on top of them.

---

## 3. Mandatory stack — never change silently

| Layer | Direction | Status |
|---|---|---|
| Runtime | Node.js + TypeScript | Confirmed |
| Framework | Next.js (App Router) + React + TypeScript | Proposed — awaiting owner sign-off |
| Hosting | Vercel | Confirmed |
| Data model | SQLite-compatible relational schema | Confirmed |
| Rendering | Public pages server-rendered / statically generated, SEO-first | Confirmed |
| ORM | Drizzle ORM + `@libsql/client` (Prisma as alternative) | Proposed |
| Production DB | Turso (libSQL) | Proposed |
| Media storage | Vercel Blob / Cloudinary / R2 / Supabase Storage | Open |

**Rule:** never silently change the Node.js runtime, the Vercel deployment target, the SQLite data-model direction, or the server-rendered SEO strategy. These are owner-level decisions. If you believe a change is needed, write the proposal into `PRD.md` (decision table + rationale + migration impact), then **wait for owner approval** before implementing. Swapping SQLite for Postgres, or moving off Vercel, is a PRD amendment — not a refactor.

Do not adopt a library, template, service, or provider that is marked *Proposed* or *Open* without explicit approval. Build against the documented **interfaces** (storage adapter, repository layer) so the choice stays reversible.

---

## 4. Hard constraints (non-negotiable)

1. **No SQLite file in production.** Vercel's runtime filesystem is read-only and ephemeral. A bundled `.db` cannot be written to and does not survive deployments or invocations. Local `file:./local.db` is **development only** and must be gitignored. Production uses the approved remote SQLite-compatible service. Any production config resolving to a `file:` URL is a release blocker.
2. **No writes to the runtime filesystem for uploads.** Media goes to object storage; the database stores an absolute public CDN URL. Never store a local path, a relative path, or a deployment-internal path.
3. **No secrets in the repository.** Credentials come from environment variables only. `.env*` files are gitignored. If a `.env.example` exists, it documents variable *names* with placeholder values and never real secrets. Never place secrets in the CMS settings table, in client bundles, or in logs.
4. **No generated media or database files committed.** No `.db`, `.sqlite`, uploads, build output, or derived image variants in Git.
5. **`logo/` is read-only reference material.** Do not edit, rename, re-encode, optimise in place, upscale, or auto-trace these files. Do not extract variants from the presentation mockup.
6. **No fabricated business facts.** Never invent certifications, testimonials, client logos, warranty periods, years of experience, project counts, awards, or partner claims. Contact details and product data come from the owner. If a value is unknown, omit the element — do not fill the gap.

---

## 5. Security expectations

- Single `Administrator` role for the MVP. Do not build RBAC, invitations, or permission matrices.
- Passwords hashed with Argon2id or bcrypt cost ≥ 12. Never reversible, never logged.
- **Every admin request re-validates the session server-side.** Middleware is a UX redirect only — it is not an authorization boundary. Each route handler and API route must independently enforce authentication.
- Session cookies: `HttpOnly`, `Secure` in production, `SameSite=Lax` or `Strict`, scoped path. No tokens in `localStorage`.
- CSRF protection on every state-changing request (origin/`Sec-Fetch-Site` check plus a per-session token).
- Login throttling: 5 failed attempts per 15 minutes per IP+account, with a generic error that does not reveal whether an account exists.
- Validate and normalise all input at boundaries: route params, query strings, form bodies, uploads, settings.
- Sanitise rich text server-side on save **and** again on render. Never render unsanitised HTML.
- Uploads: verify content type against magic bytes, enforce a size limit, strip EXIF (including GPS), reject SVG from users, and generate random opaque object keys.
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
| Integration | Publish gate validation, slug-history writes, media adapter contract |
| End-to-end | Login and logout, admin route protection when logged out, draft → preview → publish, unpublish, slug change → 301, product creation with all four price modes |
| Manual | Responsive pass at 320/360/768/1024/1280 px; keyboard-only navigation; screen-reader spot check; contrast verification |

Minimum bar for any change: it type-checks, it lints clean, and the `AC-###` it claims to satisfy is demonstrably verified.

---

## 10. Environment & deployment

- **Host:** Vercel. Production and preview deployments are separate environments with separate environment variables.
- **Region:** closest to Indonesian users and the database primary (e.g. Singapore) — to be confirmed by the owner.
- **Timezone:** `Asia/Jakarta` (UTC+7). Store timestamps in UTC; render in Jakarta time.
- **Locale:** `id-ID`. Public copy in Bahasa Indonesia; admin UI in Bahasa Indonesia.
- **Production persistence:** remote SQLite-compatible service only. The deployment filesystem is never a data store.
- **Migrations:** explicit, versioned, applied as a deliberate step. **Never auto-migrate on request in production.**
- **Revalidation:** publishing or changing settings must refresh affected public paths so content appears without a redeploy.

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
- Do not run silent or automatic schema migrations.
- Do not edit, rename, upscale, or trace the files in `logo/`.
- Do not write to the deployment filesystem for uploads or data.
- Do not commit a `.db` file or use a local SQLite file in production.
- Do not store secrets in the CMS, the repository, or client bundles.
- Do not add a second language, a language switcher, or multi-locale routing.
- Do not expose admin routes in navigation, the sitemap, or `robots.txt` allow rules.
- Do not promise SEO rankings or traffic outcomes.
- Do not reference build, lint, test, or dev commands as if they exist today.

---

## 13. Current repository state (verified 2026-10-26)

What exists:

```
AGENTS.md                  ← this file
PRD.md                     ← product requirements, source of truth
logo/
  Garda Jaya CCTV Logo Presentation.png              (presentation sheet — NOT a web asset)
  Garda-Jaya-CCTV-Main-Logo-Monochrome-Navy.png      (single-colour navy, light backgrounds/print)
  Garda-Jaya-CCTV-Main-Logo-Only.png                 (icon mark — favicon/app icon source)
  Garda-Jaya-CCTV-Main-Logo.png                      (primary lockup)
```

Git state: a `.git` directory exists (branch `main`) but it has **zero commits** and there is no `.gitignore`. The folder was not a Git repository when this documentation phase began; it was initialised during the session. Treat the repository as uncommitted and unconfigured.

What does **not** exist yet — do not assume or reference it:

- No `package.json`, no lockfile, no `node_modules`.
- No `next.config.*`, `tsconfig.json`, linter config, or test config.
- No `.env`, `.env.example`, or `.gitignore`.
- No commits — `git log` fails with "does not have any commits yet".
- No `src/`, `app/`, `public/`, `components/`, or API routes.
- No scripts, no CI pipeline, no build or dev server.

Consequences: there are **no commands to run** and **no conventions to inherit**. Setup (`.gitignore`, initial commit, project scaffold, tooling) is Phase 1 work defined in `PRD.md` §18 — not something to assume has happened. Known asset gaps (no vector master, no delivered white-on-navy variant, non-square 300×250 icon) are documented in `PRD.md` §10.2 and must not be worked around by fabricating assets.

---

*This file is durable guidance, not a copy of the PRD. When requirements change, update `PRD.md` first.*
