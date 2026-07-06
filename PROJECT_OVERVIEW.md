# Project Overview — sagarthalavar.in

A single reference doc covering all projects, pages, features, roles, and data flows.
Last updated: 2026-07-06

---

## 1. Portfolio — folio (`sagarthalavar.in`)

**Repo:** `sagar-thalavar/folio`
**Live:** https://sagarthalavar.in
**Stack:** React + TypeScript + Vite, hosted on Vercel

### Subdomains

| Subdomain | Project |
|---|---|
| `sagarthalavar.in` | Folio (main portfolio) |
| `sagarthalavar.in/guestbook` | Guestbook (proxied via Vercel rewrite, not a subdomain) |
| `collision.sagarthalavar.in` | PiCollision: Mathematical Simulator |
| `click.sagarthalavar.in` | ClickCraft: Mouse Precision Trainer |
| `data.sagarthalavar.in` | DataOps Zeta: PostgreSQL Playground |
| `equilibrium.sagarthalavar.in` | Equilibrium: Life Balance Tracker |

### Pages & Routes

| Route | Component | What it does |
|---|---|---|
| `/` | `Hero` + `Projects` + `Note` + `Footer` | Main homepage |
| `/article` | `Writings` | Blog/articles page (lazy loaded) |
| `/admin` | `Admin` | Blog post management (lazy loaded) |
| `/guestbook` | Proxied → guestbook app | Visitor journal (Vercel rewrite) |

---

### Homepage (`/`)

#### Hero section (`Hero.tsx`)
- Profile photo (`/profile.webp`), name, bio
- Bio mentions: works at BeneathAtree, 3rd-year Data Science Engineering student at New Horizon College, Bengaluru
- Kannada greeting badge: "ಸ್ವಾಗತ" (Welcome)
- Social icons — all open in a new tab (`target="_blank"`):
  - LinkedIn → `linkedin.com/in/sagar-r-thalavar-developer-gpti/`
  - GitHub → `github.com/sagar-thalavar`
  - Instagram → `instagram.com/otziburl/`
  - Email → opens Gmail compose in a new browser tab (`mail.google.com/mail/?view=cm&to=sagarthalavar509@gmail.com`)
- Two CTA buttons:
  - "Visit Guestbook" → `/guestbook`
  - "Read My Writings" → `/article` (uses SPA navigation, no full reload)

#### Live Projects section (`Projects.tsx`)
| Project | Badge | URL |
|---|---|---|
| PiCollision: Mathematical Simulator | Physics + curiosity | https://collision.sagarthalavar.in |
| ClickCraft: Mouse Precision Trainer | Skill training | https://click.sagarthalavar.in |
| DataOps Zeta: PostgreSQL Playground | Data workflow | https://data.sagarthalavar.in |
| Equilibrium: Life Balance Tracker | College mini project | https://equilibrium.sagarthalavar.in |
| Guestbook: A Visitor Journal | Web app | https://sagarthalavar.in/guestbook |

#### Temporary Note section (`Note.tsx`)
- A textarea where the user can type freely
- Does NOT save or persist anything — erased on page reload
- Has an "Erase" button to clear the text

#### Footer (`Footer.tsx`)
- Copyright line: `© [current year] · Feel free to copy anything from this page. Copyright permission is granted publicly. - Sagar`
- Year is dynamic (`new Date().getFullYear()`)

---

### Writings page (`/article`)

- Fetches published blog posts from Supabase (`posts` table, `published = true`)
- Posts sorted newest first
- Shows: title, excerpt, date, reading time, tags
- Click card or "Read Article →" to expand and read inline
- Markdown rendering support
- "← Close Article" to collapse

---

### Admin page (`/admin`) — folio blog management

- **Login:** email + password (credentials set directly in Supabase Auth, not Google OAuth)
- Only for the site owner (`sagarthalavar509@gmail.com`)
- Features:
  - **Manage tab** — list all blog posts, delete posts
  - **Editor tab** — create or edit a post with fields: title, slug, excerpt, content (markdown), tags, reading time
  - **Publish toggle** — decide if a post is live (`published = true`) or stays as a draft (`published = false`)
  - Markdown editor with live preview

---

### Global Features (folio)

| Feature | How it works |
|---|---|
| **Dark / Light theme** | Toggle button in UI; saved to `localStorage` under key `theme`; applied via `data-theme` on `<html>` |
| **Theme flash prevention** | `init.js` (external script, loads before React) reads `localStorage` and sets `data-theme` immediately so there's no flash of wrong theme on reload |
| **Non-blocking fonts** | Google Fonts loaded via `<link rel="preload">` + `font-loader.js` swaps it to stylesheet on load |
| **Code splitting** | Vite splits `@supabase`, `react/react-dom`, `lucide-react` into separate chunks; Admin and Writings are lazy-loaded |
| **CSP** | Set in `vercel.json`; `script-src 'self'` + SHA-256 hash for inline theme script; `img-src blob:` for guestbook selfie previews |
| **Security headers** | HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, COOP, CORP — all in `vercel.json` |
| **Camera permission** | `Permissions-Policy: camera=(self)` — allows guestbook selfie feature to work |
| **Email opens new tab** | Gmail compose URL instead of `mailto:` — so it opens in browser, not system mail app |

---

## 2. Guestbook (`sagarthalavar.in/guestbook`)

**Repo:** `sagar-thalavar/guestbook`
**Live:** https://sagarthalavar.in/guestbook (proxied via folio's Vercel rewrite)
**Direct Vercel URL:** https://guestbook-beta-five.vercel.app/guestbook/
**Stack:** Vanilla TypeScript + Vite, Supabase (Auth + DB + Storage), hosted on Vercel

> **Important:** Real visitors always hit `sagarthalavar.in`. Folio's `vercel.json` headers (CSP, Permissions-Policy, etc.) govern ALL requests including the proxied guestbook. Guestbook's own `vercel.json` headers only apply when accessed at its own Vercel domain directly.

---

### All Views (single `index.html`, shown/hidden by JS)

#### View 1 — Public Welcome Hero (default, visible to everyone)
- Title: "A visitor journal of moments, memories, and connections."
- Two buttons:
  - "Sign In to Leave an Entry" → goes to login view
  - "View Wall of Memories" → goes to archive view

#### View 1.5 — Wall of Memories (public archive)
- Shows all **approved + public** entries as a grid/collage
- Selfie photos served via 60-second signed URLs from Supabase Storage
- Anyone can view without signing in
- Click an entry → opens Memory Detail Modal (enlarged photo, name, mood, message, date, download button)

#### View 2 — Sign In
- **Magic Link** — user enters email → Supabase sends a login link to their inbox → click link → signed in
- **Google OAuth** — "Continue with Google" button → redirects back to `/guestbook/`
- Rate limits: 10 submissions per 24 hours per user

#### View 3 — Visitor Dashboard (signed-in users)
- Shows "Your Memories Archive" heading
- Two action cards:
  - "Create Guestbook Entry" → goes to Create Entry view
  - "View Wall of Memories" → goes to archive view
- "Your Submission Logs" — lists all of the user's own entries with their status (pending/approved/rejected)

#### View 4 — Create / Edit Entry form
Fields:
- **Name** (required)
- **Selfie photo** (optional):
  - "Start Camera" → activates device camera → "Take Photo" to capture
  - "Upload Photo Instead" → file upload fallback
  - Preview shown after capture/upload
- **Mood** (optional) — pills: Happy, Inspired, Curious, Excited, Reflective, Custom
- **Message** (optional, max 200 characters, live character counter)
- **Checkboxes:**
  - "Make this post public" — if checked and approved, entry appears on public wall
  - "I understand my submission will be reviewed" (required)
  - "I consent to storing my photo and message" (required)
- Submit button (disabled until consent boxes checked)
- Cancel button

#### View 5 — Admin Moderation Portal (admin only, hidden for regular users)
Three tabs:
- **Pending Queue** — new submissions awaiting review; badge shows count
- **All Entries** — all entries with search by name + filter by status (pending/approved/rejected) + Export to CSV button
- **Audit Logs** — table: Timestamp, Action, Target ID, Moderator, Details

Admin actions per entry: **Approve** or **Reject** (with reason)

#### Static section — Privacy & Archive Ethos
- Explains: entries are moderated, photos served via signed URLs, users have full deletion rights, public/private choice is respected

#### Navigation (top header)
- "← Back" button → returns to `sagarthalavar.in`
- **Theme toggle** — dark/light mode (same as folio, saved to `localStorage`)
- **User dropdown** (only visible when signed in):
  - "Admin Panel" — only visible to admin email
  - "Sign Out"
  - "Delete Account" (danger — permanently deletes account, all entries, all selfies)

---

### Data Flow — Guestbook

```
User arrives at sagarthalavar.in/guestbook
        ↓
Folio's Vercel rewrite proxies request → guestbook app
Folio's vercel.json headers apply (CSP, Permissions-Policy, etc.)
        ↓
User signs in (Magic Link email or Google OAuth)
        ↓
Supabase Auth creates/returns session
        ↓
Supabase DB trigger fires on auth.users:
  IF email = 'sagarthalavar509@gmail.com' → SET role = 'admin' in profiles table
        ↓
UI reads role from profiles table:
  → Admin email: shows "Admin Panel" button in dropdown
  → Regular user: "Admin Panel" button stays hidden
        ↓
User fills Create Entry form → submits
        ↓
Entry inserted into guestbook_entries (status = 'pending')
If selfie → uploaded to Supabase Storage (selfies bucket)
  Path: [userId]/[entryId]_[timestamp].jpg
  selfie_url saved to entry row
        ↓
Admin gets notified (email) → reviews in Admin Moderation Portal
        ↓
Admin approves or rejects:
  → Approved + public → entry appears on Wall of Memories
  → Rejected → user can edit and resubmit (max 3 attempts)
        ↓
Public Wall fetches entries (status=approved, is_public=true)
For each entry with selfie → generates 60-second signed URL on render
```

---

### Roles & Permissions

| Role | Who | Privileges |
|---|---|---|
| **Anonymous** | Not signed in | View public wall, view memory detail modal |
| **User** | Signed in (any email) | All anonymous + submit entries, view own entries, edit/resubmit rejected entries, delete own account |
| **Admin** | `sagarthalavar509@gmail.com` | All user privileges + approve/reject any entry, view all entries, view audit logs, export CSV, bypass rate limits, admin role set automatically via DB trigger |

### Rate Limits
- Max **10 submissions per 24 hours** per user
- Admin bypasses all rate limit checks

### Entry Status Flow
```
submitted → pending → approved (appears on public wall if is_public=true)
                   ↘ rejected → user edits → pending again (max 3 re-attempts)
```

### Selfie Storage
- Bucket: `selfies` (private)
- Path format: `[userId]/[entryId]_[timestamp].jpg`
- Served only via 60-second signed URLs — never directly exposed

---

## 3. GitHub Profile README (`sagar-thalavar/sagar-thalavar`)

**Live:** https://github.com/sagar-thalavar

### Contents
- Bio: Data Science Engineering student (3rd Year), Web Developer at BeneathAtree
- Badges: Website, LinkedIn, Email
- Tech stack table: Frontend (React, TypeScript, JS, HTML, CSS, Vite) + Backend (Node.js, Python, Git, PostgreSQL)
- **Highlighted Work** list (Guestbook is first):
  1. Guestbook
  2. PI Collision Simulator
  3. Mouse Practice
  4. DataOps Zeta
  5. Equilibrium
- GitHub stats widgets (stats card + streak card, tokyonight theme)
- Quote: *"Simple, pleasant to use, and easy to understand."*
- `STREAK.md` — auto-updated daily by cron workflow
- `future plans.md` — list of features to build
- `PROJECT_OVERVIEW.md` — this file

### Daily Streak Workflow (`.github/workflows/daily-streak.yml`)

| Setting | Value |
|---|---|
| **Trigger** | Scheduled: 03:00 UTC daily + manual `workflow_dispatch` |
| **Random delay** | 0–16 hours sleep before committing (schedule only, skipped on manual) |
| **Commit count** | Random 1–5 commits per day |
| **Commit gap** | Random 0–5 min between commits (schedule) / max 10s (manual, for testing) |
| **What it commits** | Appends timestamp line to `STREAK.md` |
| **Permission** | `contents: write` declared in workflow — no repo settings change needed |
| **Branch protection** | Ruleset removed from this repo — bot pushes directly to main |
| **No laptop needed** | Runs entirely on GitHub's servers |

---

## 4. Automation & Testing (folio)

**Folder:** `sagar-thalavar/folio/tests/`

| Test file | What it tests |
|---|---|
| `tests/smoke.spec.ts` | Homepage loads with no console errors; theme toggle works (full round-trip: click → different → click → back to original); Writings route loads without errors |
| `tests/security-headers.spec.ts` | Reads `vercel.json` directly (no browser needed) — checks: Permissions-Policy allows `camera=(self)`, CSP `img-src` has `blob:`, CSP SHA-256 hash matches actual inline script in `dist/index.html` |

**Why no-browser tests for headers:** `vite preview` (local dev server) never sends `vercel.json` headers — CSP bugs are invisible in local Playwright tests. Reading the config file directly catches mismatches before deploy.

**CI Pipeline** (`.github/workflows/test.yml`) — runs on every push and PR to main:
1. Checkout + install Node 20
2. Install Playwright + Chromium
3. Build (with Supabase env secrets from GitHub Secrets)
4. Run Playwright e2e tests
5. Run Lighthouse CI (thresholds: performance ≥ 0.85, accessibility ≥ 0.90, best practices ≥ 0.90, SEO ≥ 0.90)
6. Upload Playwright report as artifact on failure (7-day retention)

**Dependabot** (`.github/dependabot.yml`): Weekly npm + GitHub Actions updates, grouped by production/dev deps.

---

## 5. Branch Protection (GitHub Rulesets)

Applied to 10 public repos. Rules per repo:
- Direct push to main blocked — must go through a PR
- No force push to main
- No branch deletion

**Exception:** `sagar-thalavar/sagar-thalavar` (profile README) — ruleset removed so the daily streak bot can push directly to main.

**Private repos** (Beneathatree-handbook-portal, MysteryButton, Voucher) — rulesets require GitHub Pro for private repos, not applied.

---

## 6. Key Infrastructure Notes

| Topic | Detail |
|---|---|
| **Folio proxies guestbook** | `vercel.json` rewrites `/guestbook/*` → `guestbook-beta-five.vercel.app/guestbook/*`. Folio's headers govern all requests including proxied guestbook. |
| **CSP blind spot** | `vite preview` never sends `vercel.json` headers. `security-headers.spec.ts` catches header regressions by reading config directly. |
| **Supabase** | Both folio and guestbook use Supabase. Folio: blog posts (`posts` table). Guestbook: entries, profiles, selfie storage, audit logs. |
| **Admin role assignment** | Set via Supabase Postgres trigger on `auth.users` — server-side only, cannot be spoofed from frontend. |
| **Selfie privacy** | Supabase Storage `selfies` bucket is private. All photos served via short-lived signed URLs (60s TTL). |
| **Theme persistence** | Both folio and guestbook save theme to `localStorage`. Folio uses `init.js` to apply before React loads (prevents flash). |
