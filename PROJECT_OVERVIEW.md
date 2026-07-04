# Project Overview — sagarthalavar.in

A single reference doc covering all projects, pages, features, roles, and data flows.
Last updated: 2026-07-05

---

## 1. Portfolio — folio (`sagarthalavar.in`)

**Repo:** `sagar-thalavar/folio`
**Live:** https://sagarthalavar.in
**Stack:** React + TypeScript + Vite, hosted on Vercel

### Pages & Routes

| Route | Component | What it does |
|---|---|---|
| `/` | `Hero` + `Projects` + `Note` + `Footer` | Main homepage |
| `/article` | `Writings` | Blog/articles page (lazy loaded) |
| `/admin` | `Admin` | Blog post management (lazy loaded) |
| `/guestbook` | Proxied → guestbook app | Visitor journal (Vercel rewrite) |

### Homepage (`/`)

**Hero section** (`Hero.tsx`):
- Profile photo, name, bio
- Social icons: LinkedIn, GitHub, Instagram, Email
  - Email opens Gmail compose in a new tab (`mail.google.com/mail/?view=cm&to=...`)
  - All social links open in a new tab (`target="_blank"`)
- Two CTA buttons: "Visit Guestbook" → `/guestbook`, "Read My Writings" → `/article`

**Live Projects section** (`Projects.tsx`):
| Project | URL |
|---|---|
| PiCollision: Mathematical Simulator | https://collision.sagarthalavar.in |
| ClickCraft: Mouse Precision Trainer | https://click.sagarthalavar.in |
| DataOps Zeta: PostgreSQL Playground | https://data.sagarthalavar.in |
| Equilibrium: Life Balance Tracker | https://equilibrium.sagarthalavar.in |
| Guestbook: A Visitor Journal | https://sagarthalavar.in/guestbook |

**Note section** (`Note.tsx`): A short personal note/message.

**Footer** (`Footer.tsx`): Copyright and links.

### Writings page (`/article`)

- Fetches published blog posts from Supabase (`posts` table, `published = true`)
- Shows title, excerpt, date, reading time, tags
- Click to expand and read the full article inline
- Markdown rendering support

### Admin page (`/admin`)

- Password-protected (email + password login via Supabase Auth)
- Only accessible to the account owner
- Features:
  - Create, edit, delete blog posts
  - Toggle published/draft status
  - Markdown editor with live preview
  - Manage tab + Editor tab

### Global Features

| Feature | How it works |
|---|---|
| **Dark / Light theme** | Toggle button in UI; preference saved to `localStorage` under key `theme`; applied instantly via `data-theme` attribute on `<html>` |
| **Theme persistence** | `init.js` runs before React loads — reads `localStorage` and sets `data-theme` so there's no flash of wrong theme on reload |
| **Non-blocking fonts** | Google Fonts loaded via `<link rel="preload">` + `font-loader.js` swaps it to stylesheet on load — does not block page render |
| **Code splitting** | Vite splits `@supabase`, `react/react-dom`, and `lucide-react` into separate chunks; Admin and Writings routes are lazy-loaded |
| **CSP (Content Security Policy)** | Set in `vercel.json`; `script-src 'self'` + SHA-256 hash for inline theme script; `img-src blob:` for guestbook selfie previews |
| **Security headers** | HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, COOP, CORP all set in `vercel.json` |
| **Camera permission** | `Permissions-Policy: camera=(self)` — allows camera access for guestbook selfie feature |

### Data Flow — folio

```
User visits sagarthalavar.in
        ↓
Vercel serves index.html
        ↓
init.js runs → sets data-theme from localStorage (prevents theme flash)
        ↓
React loads → Hero, Projects, Note, Footer render
        ↓
If user visits /article or /admin → lazy chunk loads → Supabase query
        ↓
If user visits /guestbook → Vercel rewrites request → guestbook app
```

---

## 2. Guestbook (`sagarthalavar.in/guestbook`)

**Repo:** `sagar-thalavar/guestbook`
**Live:** https://sagarthalavar.in/guestbook (proxied via folio's Vercel rewrite)
**Own Vercel domain:** https://guestbook-beta-five.vercel.app/guestbook/
**Stack:** Vanilla TypeScript + Vite, Supabase (Auth + DB + Storage), hosted on Vercel

> **Important:** Real visitors always hit the folio domain (`sagarthalavar.in`). Folio's `vercel.json` headers (CSP, Permissions-Policy, etc.) govern ALL requests including the proxied guestbook. Guestbook's own `vercel.json` headers only apply when accessed at its own Vercel domain directly.

### Pages / Views (all in a single `index.html`, shown/hidden by JS)

| View | Who sees it | What it does |
|---|---|---|
| **Landing / Sign-in** | Everyone | Google OAuth + Magic Link email sign-in |
| **Dashboard** | Signed-in users | Shows user's own entries, lets them write a new entry |
| **New Entry form** | Signed-in users | Name, message, mood, selfie/photo upload, public/private toggle |
| **Public Wall** | Everyone | Shows all approved + public entries |
| **Admin Moderation Portal** | Admin only | Approve/reject entries, view audit logs, export CSV |

### Features

| Feature | Details |
|---|---|
| **Google OAuth sign-in** | Via Supabase Auth |
| **Magic Link sign-in** | Email OTP — sends a link to sign in without password |
| **Selfie capture** | Uses browser camera (`getUserMedia`) — requires `camera=(self)` in Permissions-Policy |
| **Photo upload** | Upload from device as alternative to selfie |
| **Selfie preview** | Uses `URL.createObjectURL(blob)` — requires `img-src blob:` in CSP |
| **Signed selfie URLs** | Selfies stored privately in Supabase Storage; 60-second signed URLs generated per view |
| **Entry status flow** | `pending` → `approved` or `rejected` |
| **Re-upload on rejection** | Users can edit and resubmit rejected entries (max 3 attempts) |
| **Rate limiting** | Max 10 submissions per 24 hours per user |
| **Public/private toggle** | Users choose if their entry shows on the public wall |
| **Mood selector** | Users pick a mood when submitting |
| **Admin auto-approve** | Admin account (`sagarthalavar509@gmail.com`) gets `role = 'admin'` via Supabase DB trigger — bypasses rate limits and all manual approval |
| **Audit logs** | Admin can view a log of all moderation actions |
| **CSV export** | Admin can export all entries to CSV |
| **Account deletion** | Users can permanently delete their account + all entries + selfies |

### Roles

| Role | Who | Privileges |
|---|---|---|
| **Anonymous** | Not signed in | View public wall only |
| **User** | Signed in via Google/Magic Link | Submit entries, view own entries, delete own account |
| **Admin** | `sagarthalavar509@gmail.com` | All user privileges + approve/reject any entry, view all entries, view audit logs, export CSV, bypass rate limits |

### Data Flow — Guestbook

```
User signs in (Google OAuth or Magic Link)
        ↓
Supabase Auth creates/returns session
        ↓
DB trigger checks email → if admin email, sets role = 'admin' in profiles table
        ↓
UI checks role → shows/hides Admin Panel button
        ↓
User submits entry
        ↓
Entry saved to guestbook_entries (status = 'pending')
If selfie → uploaded to Supabase Storage (selfies bucket) → selfie_url saved
        ↓
Admin reviews → approves or rejects
        ↓
If approved + public → appears on Public Wall
Public Wall fetches signed URLs for each selfie on render
```

---

## 3. GitHub Profile README (`sagar-thalavar/sagar-thalavar`)

**Live:** https://github.com/sagar-thalavar

### Features

| Feature | Details |
|---|---|
| **Highlighted Work list** | Links to all 5 live projects with descriptions; Guestbook is first |
| **Tech stack table** | Frontend and backend technologies with badge icons |
| **GitHub stats** | github-readme-stats widget showing commits, streak |
| **Daily streak cron** | GitHub Actions workflow runs every day at 03:00 UTC |

### Daily Streak Workflow (`.github/workflows/daily-streak.yml`)

| Setting | Value |
|---|---|
| **Trigger** | Scheduled: 03:00 UTC daily + manual `workflow_dispatch` |
| **Random delay** | 0–16 hours sleep before committing (schedule only, skipped on manual) |
| **Commit count** | Random 1–5 commits per day |
| **Commit gap** | Random 0–5 min between commits (schedule) / max 10s (manual) |
| **What it commits** | Appends timestamp to `STREAK.md` |
| **Why** | Keeps GitHub contribution graph green every day automatically |
| **No laptop needed** | Runs entirely on GitHub's servers |
| **Branch protection** | Ruleset removed from this repo so the bot can push directly to main |

### Data Flow — Streak

```
03:00 UTC alarm fires
        ↓
Random sleep (0–16 hrs) → actual commit time is unpredictable
        ↓
Pick random count (1–5)
        ↓
For each: sleep random gap → append timestamp to STREAK.md → git commit
        ↓
git push → commits land on main → GitHub contribution graph updates
```

---

## 4. Automation & Testing (folio)

**Repo:** `sagar-thalavar/folio` → `tests/` folder

| Test file | What it tests |
|---|---|
| `tests/smoke.spec.ts` | Homepage loads, no console errors, theme toggle works (full round-trip), Writings route loads |
| `tests/security-headers.spec.ts` | Reads `vercel.json` directly (no browser) — checks Permissions-Policy camera, CSP `img-src blob:`, CSP hash matches inline script in `dist/index.html` |

**CI:** GitHub Actions (`.github/workflows/test.yml`) runs on every push and PR:
1. Install deps + Playwright
2. Build (with Supabase env secrets)
3. Run Playwright e2e tests
4. Run Lighthouse CI (performance ≥ 0.85, accessibility ≥ 0.90, best practices ≥ 0.90, SEO ≥ 0.90)
5. Upload Playwright report as artifact on failure

**Dependabot:** Weekly npm + GitHub Actions dependency updates, grouped by production/dev.

---

## 5. Branch Protection (GitHub Rulesets)

Applied to 10 repos (all public project repos). Rules:
- No direct push to main — must go through a PR
- No force push
- No branch deletion

**Exception:** `sagar-thalavar/sagar-thalavar` (profile README) has no ruleset so the daily streak bot can push directly to main.

---

## 6. Key Infrastructure Notes

| Topic | Detail |
|---|---|
| **Folio proxies guestbook** | `vercel.json` rewrites `/guestbook/*` → `guestbook-beta-five.vercel.app/guestbook/*`. This means folio's CSP/headers govern guestbook for real visitors. |
| **CSP blind spot** | `vite preview` (local dev) never sends `vercel.json` headers — CSP bugs are invisible locally. The `security-headers.spec.ts` test catches these by reading the config file directly. |
| **Supabase** | Both folio and guestbook share Supabase for auth and DB. Folio uses it for blog posts. Guestbook uses it for entries, profiles, selfie storage, and audit logs. |
| **Admin role** | Set via a Supabase Postgres trigger server-side — not in frontend code. Cannot be spoofed or manipulated from the browser. |
