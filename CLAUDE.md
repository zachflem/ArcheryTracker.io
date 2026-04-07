# ArcheryTracker.io — Claude Code Project Prompt

## Your Role

You are the lead engineer on **ArcheryTracker.io**, a mobile-first progressive web app that replaces the paper scorecard used in field archery competitions. You have full context of the product architecture, data model, and business rules. Build thoughtfully, ask before making significant structural decisions, and always keep the MVP scope in view.

---

## Repository & Infrastructure

- **GitHub repo:** `zachflem/ArcheryTracker.io` — already initialised and ready
- **Cloudflare integrations in use:** Pages (frontend hosting), Workers (API), D1 (database), KV (tokens + cache)
- **Cloudflare Pages** is connected to the GitHub repo — pushes to `main` auto-deploy
- **Node.js** package manager: `pnpm` with workspaces (monorepo)
- All Cloudflare resources should be configured via `wrangler.toml` — do not use the dashboard for resource creation where CLI is available

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + TypeScript, Vite, Tailwind CSS |
| PWA / Offline | `vite-plugin-pwa` + Workbox (service worker, background sync) |
| Local storage | `idb` (typed IndexedDB wrapper) |
| Backend | Cloudflare Worker, Hono router |
| Database | Cloudflare D1 (SQLite) |
| Cache / tokens | Cloudflare KV |
| Auth | Magic-link email, JWT (`jose`), HttpOnly cookie |
| Email | Resend (outbound transactional) |
| QR scanning | `@zxing/browser` |
| QR generation | `qrcode` |
| Validation | `zod` (API inputs) |
| Monorepo | pnpm workspaces |

---

## Monorepo Structure

Scaffold the repository exactly as follows:

```
archerytracker/
├── apps/
│   ├── web/                          # React PWA → Cloudflare Pages
│   │   ├── src/
│   │   │   ├── pages/
│   │   │   │   ├── Login.tsx
│   │   │   │   ├── Dashboard.tsx
│   │   │   │   ├── Session/
│   │   │   │   │   ├── Setup.tsx
│   │   │   │   │   ├── Scoring.tsx
│   │   │   │   │   └── Summary.tsx
│   │   │   │   ├── History.tsx
│   │   │   │   ├── Card.tsx
│   │   │   │   └── club/             # [v2] — stub files only
│   │   │   │       ├── Landing.tsx
│   │   │   │       ├── AdminDash.tsx
│   │   │   │       └── CourseBuilder.tsx
│   │   │   ├── components/
│   │   │   │   ├── OrgPicker.tsx
│   │   │   │   ├── RoundPicker.tsx
│   │   │   │   ├── ShooterAdd.tsx
│   │   │   │   ├── ScoreInput.tsx
│   │   │   │   ├── StopOnHitFeedback.tsx
│   │   │   │   ├── IndoorTimer.tsx
│   │   │   │   ├── PostTargetReveal.tsx
│   │   │   │   ├── Scorecard.tsx
│   │   │   │   └── QRCode.tsx
│   │   │   ├── lib/
│   │   │   │   ├── db.ts             # IndexedDB schema + queries
│   │   │   │   ├── sync.ts           # Offline mutation queue
│   │   │   │   ├── api.ts            # Typed fetch wrapper
│   │   │   │   ├── scoring.ts        # Score computation + grading bands
│   │   │   │   └── units.ts          # Yards ↔ metres display
│   │   │   └── sw.ts                 # Workbox service worker entry
│   │   ├── index.html
│   │   ├── vite.config.ts
│   │   ├── tailwind.config.ts
│   │   └── tsconfig.json
│   │
│   └── worker/                       # Cloudflare Worker → API
│       ├── src/
│       │   ├── index.ts              # Hono app entry + route registration
│       │   ├── routes/
│       │   │   ├── auth.ts
│       │   │   ├── disciplines.ts
│       │   │   ├── sessions.ts
│       │   │   ├── scores.ts
│       │   │   ├── shooters.ts
│       │   │   └── clubs.ts          # [v2] — stub only
│       │   ├── lib/
│       │   │   ├── db.ts             # D1 query helpers
│       │   │   ├── email.ts          # Resend integration
│       │   │   ├── jwt.ts            # jose sign/verify
│       │   │   ├── scorecard.ts      # HTML email renderer
│       │   │   └── permissions.ts    # Auth middleware + role checks
│       │   └── seed/
│       │       ├── aba-disciplines.ts
│       │       └── ifaa-disciplines.ts
│       ├── wrangler.toml
│       └── tsconfig.json
│
├── packages/
│   └── shared/
│       └── src/
│           ├── types.ts              # Shared TypeScript interfaces
│           ├── discipline-rules.ts   # Zone/scoring logic (web + worker)
│           └── units.ts              # Conversion constants
│
├── pnpm-workspace.yaml
├── package.json
└── tsconfig.base.json
```

---

## Database Schema

Create this schema in `apps/worker/src/schema.sql`. Apply it to D1 using `wrangler d1 execute`. Do not skip the v2-stubbed tables — they must exist from day one so v2 requires no migrations.

```sql
-- ─── USERS ────────────────────────────────────────────────────────────────
CREATE TABLE users (
  id                    TEXT PRIMARY KEY,
  email                 TEXT UNIQUE NOT NULL,
  display_name          TEXT NOT NULL,
  membership_id         TEXT UNIQUE NOT NULL,
  role                  TEXT NOT NULL DEFAULT 'shooter',
  preferred_aba_style   TEXT,
  preferred_ifaa_style  TEXT,
  age_division          TEXT NOT NULL DEFAULT 'senior',
  created_at            INTEGER NOT NULL
);

-- ─── CLUBS (v2 stub — create table now, populate in v2) ───────────────────
CREATE TABLE clubs (
  id              TEXT PRIMARY KEY,
  slug            TEXT UNIQUE NOT NULL,
  name            TEXT NOT NULL,
  address         TEXT,
  lat             REAL,
  lng             REAL,
  contact_email   TEXT,
  contact_phone   TEXT,
  website         TEXT,
  is_published    INTEGER NOT NULL DEFAULT 0,
  created_at      INTEGER NOT NULL
);

CREATE TABLE club_admins (
  user_id         TEXT NOT NULL REFERENCES users(id),
  club_id         TEXT NOT NULL REFERENCES clubs(id),
  granted_at      INTEGER NOT NULL,
  PRIMARY KEY (user_id, club_id)
);

-- ─── DISCIPLINES (seeded, site_admin only) ────────────────────────────────
CREATE TABLE disciplines (
  id              TEXT PRIMARY KEY,
  organisation    TEXT NOT NULL,
  name            TEXT NOT NULL,
  short_name      TEXT NOT NULL,
  category        TEXT NOT NULL,
  target_count    INTEGER NOT NULL,
  round_structure TEXT NOT NULL,
  score_zones     TEXT NOT NULL,
  distance_table  TEXT,
  grading_table   TEXT,
  special_rules   TEXT NOT NULL,
  distance_unit   TEXT NOT NULL DEFAULT 'metric',
  created_at      INTEGER NOT NULL
);

-- ─── COURSES (v2 stub) ────────────────────────────────────────────────────
CREATE TABLE courses (
  id              TEXT PRIMARY KEY,
  club_id         TEXT NOT NULL REFERENCES clubs(id),
  discipline_id   TEXT NOT NULL REFERENCES disciplines(id),
  name            TEXT NOT NULL,
  target_count    INTEGER NOT NULL,
  is_published    INTEGER NOT NULL DEFAULT 0,
  last_updated    INTEGER NOT NULL,
  created_at      INTEGER NOT NULL
);

CREATE TABLE course_targets (
  id              TEXT PRIMARY KEY,
  course_id       TEXT NOT NULL REFERENCES courses(id),
  target_number   INTEGER NOT NULL,
  group_series    TEXT,
  face_size       TEXT,
  distance_m      REAL,
  distance_yds    REAL,
  marker_count    INTEGER NOT NULL DEFAULT 1,
  notes           TEXT,
  UNIQUE(course_id, target_number)
);

-- ─── SESSIONS ─────────────────────────────────────────────────────────────
CREATE TABLE sessions (
  id              TEXT PRIMARY KEY,
  discipline_id   TEXT NOT NULL REFERENCES disciplines(id),
  course_id       TEXT REFERENCES courses(id),
  owner_id        TEXT NOT NULL REFERENCES users(id),
  club_tag        TEXT,
  is_grading      INTEGER NOT NULL DEFAULT 0,
  grading_locked  INTEGER NOT NULL DEFAULT 0,
  started_at      INTEGER NOT NULL,
  completed_at    INTEGER,
  status          TEXT NOT NULL DEFAULT 'active'
);

-- ─── SESSION SHOOTERS ─────────────────────────────────────────────────────
CREATE TABLE session_shooters (
  id              TEXT PRIMARY KEY,
  session_id      TEXT NOT NULL REFERENCES sessions(id),
  user_id         TEXT REFERENCES users(id),
  guest_email     TEXT,
  display_name    TEXT NOT NULL,
  age_division    TEXT NOT NULL DEFAULT 'senior',
  shooting_style  TEXT,
  joined_at       INTEGER NOT NULL
);

-- ─── SCORES ───────────────────────────────────────────────────────────────
CREATE TABLE scores (
  id              TEXT PRIMARY KEY,
  session_id      TEXT NOT NULL REFERENCES sessions(id),
  shooter_id      TEXT NOT NULL REFERENCES session_shooters(id),
  target_number   INTEGER NOT NULL,
  arrow_number    INTEGER NOT NULL,
  marker_position INTEGER,
  zone_label      TEXT NOT NULL,
  value           INTEGER NOT NULL,
  recorded_at     INTEGER NOT NULL,
  synced          INTEGER NOT NULL DEFAULT 1
);
```

---

## Authentication

Magic-link, passwordless. No passwords stored anywhere.

**Flow:**
1. User submits email to `POST /auth/request`
2. Worker generates a cryptographically random token, stores in KV with TTL 15 minutes, key: `magic:<token>`
3. Worker sends email via Resend: `https://archerytracker.io/auth/verify?token=<token>`
4. User clicks → `POST /auth/verify` → Worker validates token, deletes from KV, creates or retrieves user, issues signed JWT in HttpOnly cookie
5. JWT payload: `{ userId, email, displayName, membershipId, role }`
6. JWT expiry: 30 days

**Guest invite flow:**
- Scorecard emails to non-registered guests include: `/invite?ref=<sessionId>&email=<b64email>&token=<inviteToken>`
- Invite tokens in KV, key: `invite:<token>`, TTL: 7 days
- On first login via invite link, guest scores from that session are claimed to the new account

---

## API Endpoints (MVP)

All routes except `/auth/*` and `/shooter/:membershipId` require a valid JWT cookie.

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/auth/request` | None | Send magic link |
| POST | `/auth/verify` | None | Verify token, set JWT cookie |
| GET | `/me` | Required | User profile |
| GET | `/me/card` | Required | Membership card data |
| PATCH | `/me` | Required | Update display name, preferred styles, age division |
| GET | `/shooter/:membershipId` | None (rate-limited) | QR shooter lookup |
| GET | `/disciplines` | None | All disciplines grouped by org. KV-cached 24hr. |
| POST | `/sessions` | Required | Create session |
| GET | `/sessions/:id` | Required | Session + shooters + scores |
| PATCH | `/sessions/:id` | Required (owner) | Update club tag or grading flag (locked after first score) |
| POST | `/sessions/:id/shooters` | Required | Add shooter (registered or guest) |
| PATCH | `/sessions/:id/shooters/:shooterId` | Required | Update shooter style/division for this session |
| POST | `/sessions/:id/scores` | Required | Record scores — accepts batch array for offline flush |
| POST | `/sessions/:id/complete` | Required (owner) | Finalise round, trigger emails |
| GET | `/sessions/:id/scorecard` | Required | Full scorecard data |
| GET | `/history` | Required | Authenticated user's session history |

---

## Discipline Seed Data

The `disciplines` table is seeded once. Each discipline's `special_rules` JSON **must** include a `distance_visibility` field:
- `"always"` — shown throughout (Indoor rounds)
- `"after_target"` — hidden during scoring, revealed on post-target screen and final scorecard
- `"never"` — never shown (3D and Animal Unmarked rounds)

### ABA Disciplines

| ID | Name | Targets | Arrows | Diminishing | Stop-on-hit | Timed | Distance Visibility |
|----|------|---------|--------|-------------|-------------|-------|---------------------|
| `aba-field-3arrow` | National Field Round — 3 Arrow | 20 | 3 | ✅ | ❌ | ❌ | after_target |
| `aba-field-1arrow` | National Field Round — 1 Arrow | 20 | 1 | ❌ | ❌ | ❌ | after_target |
| `aba-3d-3arrow` | National 3D Round — 3 Arrow | 20 | 3 | ✅ | ❌ | ❌ | never |
| `aba-3d-2arrow` | National 3D Round — 2 Arrow | 20 | 2 | ✅ | ❌ | ❌ | never |
| `aba-3d-1arrow` | National 3D Round — 1 Arrow | 20 | 1 | ❌ | ❌ | ❌ | never |
| `aussie-standard` | Aussie Field Round — Standard | 20 | 3 | ❌ | ❌ | ❌ | after_target |
| `aussie-expert` | Aussie Field Round — Expert | 20 | varies by series | ❌ | ❌ | ❌ | after_target |
| `aba-indoor` | National Indoor Round | 4 games | 5/game | ❌ | ❌ | ✅ 5min/game | always |

**ABA Kill/Vital/Wound zones (diminishing):**
- Arrow 1: A=20, B=18, C=16, Miss=0
- Arrow 2: A=14, B=12, C=10, Miss=0
- Arrow 3: A=8, B=6, C=4, Miss=0

**ABA Aussie Standard:** Bull=5, Inner=4, Outer=3, Miss=0 (all arrows equal)

**ABA Aussie Expert:** Bull Inner=5, Bull Outer=4, Inner Colour Inner=3, Inner Colour Outer=2, Outer Colour=1, Miss=0

**ABA Indoor:** A=20, B=18, C=16, Miss=0 (not diminishing)

**ABA Aussie Expert arrow count by series:** Series 1=1, Series 2=2, Series 3=3, Series 4=4, Series 5=5

### IFAA Disciplines (MVP: 4 rounds only)

| ID | Name | Targets | Arrows | Diminishing | Stop-on-hit | Timed | Distance Visibility |
|----|------|---------|--------|-------------|-------------|-------|---------------------|
| `ifaa-field` | IFAA Field Round | 28 | 4 | ❌ | ❌ | ❌ | after_target |
| `ifaa-hunter` | IFAA Hunter Round | 28 | 4 | ❌ | ❌ | ❌ | after_target |
| `ifaa-animal-unmarked` | IFAA Animal Round — Unmarked | 28 | 1–3 | ✅ | ✅ (reminder) | ❌ | never |
| `ifaa-indoor` | IFAA Indoor Round | 12 ends | 5/end | ❌ | ❌ | ✅ 4min/end | always |

**IFAA Field/Hunter zones (all arrows equal):** 5=Spot, 4=Inner Ring, 3=Outer Ring, Miss=0

**IFAA Animal zones (diminishing):**
- Arrow 1: Kill=20, Wound=18, Miss=0
- Arrow 2: Kill=16, Wound=14, Miss=0
- Arrow 3: Kill=12, Wound=10, Miss=0

**IFAA Indoor zones:** X=5 (tiebreak flag), Spot=5, 4, 3, 2, 1, Miss=0

**Distance units:** IFAA = `imperial` (yards first: `50 yds / 45.7m`). ABA = `metric` (metres first: `36m / 39.4yds`).

---

## Scorecard UI Rules

The `ScoreInput` component must be entirely config-driven. Zero discipline-specific branching inside any component.

**Per-target screen shows:**
- Target number + total (e.g. "Target 7 of 20")
- Running total score
- One arrow prompt at a time — next arrow only appears after current is scored
- For diminishing disciplines: show max value for current arrow (e.g. "Arrow 2 — max 14 pts")
- Large tap zone buttons — minimum 56px height, full-width on mobile
- "Miss" button always present, visually de-emphasised
- "← Back" to correct previous target, "Next →" to advance (only enabled once all arrows for current target are recorded)

**Stop-on-hit (IFAA Animal):**
- After any scoring zone is tapped, remaining arrow rows collapse
- Non-blocking banner: "Hit recorded — move to next target"
- No modal. Scorer taps "Next →" freely.

**Indoor timer:**
- ABA Indoor: 5-minute countdown per game
- IFAA Indoor: 4-minute countdown per end
- At 30 seconds: timer turns amber
- At 0 seconds: timer turns red, device vibrates (if permitted), alert sound plays
- Timer does NOT auto-advance. Scorer manually taps to close the end.

**Distance visibility:**
- `always`: distance label shown above zone buttons throughout
- `after_target`: hidden during arrow entry. After "Next →", a `PostTargetReveal` screen shows the distance. Dismisses on tap or auto after 2 seconds.
- `never`: distance never shown anywhere — not during scoring, not in summary, not in email

---

## Offline-First Architecture

The app must work fully after a single online session.

**IndexedDB** (`idb`) is the source of truth for active sessions:
- Session created → written to IndexedDB immediately
- Score recorded → written to IndexedDB first, then attempted to Worker API
- If offline → added to `pendingSync` queue in IndexedDB; `synced = 0` on score record

**Workbox Background Sync:**
- Queue name: `score-sync`
- On reconnect: replays queued requests in order
- Retry with exponential backoff up to 48 hours

**Caching:**
- App shell (HTML/JS/CSS): cache on first load
- `/disciplines` response: cache 24 hours
- All other API routes: network-first, cache as fallback

**Conflict resolution:** Last-write-wins per `(session_id, shooter_id, target_number, arrow_number)` using `recorded_at`.

---

## QR Code System

**Membership card QR** encodes: `https://archerytracker.io/shooter/<membershipId>`

`membershipId` is a UUID, stable and never changes.

**Scanning:**
1. Read QR → extract `membershipId` → call `GET /shooter/<membershipId>`
2. Online: returns `{ displayName, membershipId, email }` → add to session
3. Offline: store URL in IndexedDB as pending shooter; resolve on sync

**Membership card (`/card`):**
- Auth-required
- Shows: display name, membership ID, QR code
- `@media print` hides all nav — card fills the page
- User prints or saves as PDF from browser

---

## Shooting Style Rules

**Profile level** (`users.preferred_aba_style`, `users.preferred_ifaa_style`):
- Default pre-fill only when joining a session
- Editing these never retroactively changes past sessions

**Session level** (`session_shooters.shooting_style`):
- Pre-filled from profile preference at join time
- Always editable by the shooter — for any round, at any time before completion
- Completely independent of profile preference

**Grading enforcement:**
- If `is_grading = 1` and `grading_locked = 0`: all shooters must have `shooting_style` and `age_division` set before first score is accepted
- Once any score is recorded: `grading_locked = 1` — API rejects further changes to `is_grading`

**Valid ABA styles:** `BH_C`, `BH_R`, `TLB`, `MLB`, `HB`, `FS_R`, `FS_C`, `FU`, `BU`, `BL`, `TRAD_PEG_LB`, `TRAD_PEG_MLB`, `TRAD_PEG_R`, `TRAD_PEG_HB`

**Valid IFAA styles:** `BB_R`, `BB_C`, `FS_R`, `FS_C`, `FU`, `BH_R`, `BH_C`, `BL`, `BU`, `LB`, `HB`, `TR`

**Valid age divisions:** `senior`, `veteran`, `adult`, `young_adult`, `junior`, `cub`

---

## Scorecard Email

Triggered by `POST /sessions/:id/complete` via Resend.

- All styles inline (no `<style>` blocks — stripped by Gmail)
- Subject: `[ArcheryTracker] Your scorecard — {discipline} · {club_tag or 'Open Round'} · {date}`
- Includes all shooters' scores in a table + target-by-target breakdown for the recipient
- Marked rounds (`distance_visibility = 'after_target'`): distances shown in email table
- Unmarked rounds (`distance_visibility = 'never'`): no distances anywhere in email
- Guest shooters receive scorecard + invite section: "Join ArcheryTracker — [Create account]" → `/invite?ref=<sessionId>&email=<b64>&token=<inviteToken>`
- Invite tokens: KV key `invite:<token>`, TTL 7 days

---

## Wrangler Configuration

`apps/worker/wrangler.toml`:
- Worker name: `archerytracker-api`
- D1 binding: `DB`
- KV binding: `KV`
- Secrets (via `wrangler secret put`, not in toml): `RESEND_API_KEY`, `JWT_SECRET`
- Route: `archerytracker.io/api/*` → Worker

Cloudflare Pages project: `archerytracker-web`, linked to `zachflem/ArcheryTracker.io` on `main` branch.

---

## Build Order

### Phase 1 — Foundation (complete each before moving on)

1. **Repo scaffold** — initialise pnpm workspaces, all directories and stub files per structure above, install all dependencies, confirm TypeScript compiles clean across all packages
2. **D1 schema** — `wrangler d1 create archerytracker-db`, apply `schema.sql`, verify all tables exist
3. **Shared types** — define all TypeScript interfaces in `packages/shared/src/types.ts`: `User`, `Session`, `SessionShooter`, `Score`, `Discipline`, `ScoreZone`, `RoundStructure`, `SpecialRules`
4. **Discipline seed** — implement `aba-disciplines.ts` and `ifaa-disciplines.ts` with full seed objects for all 12 disciplines; write `seed.ts` script; run and verify
5. **Auth** — `POST /auth/request`, `POST /auth/verify`, Resend magic-link email, JWT issue and validation; test with a real email
6. **User profile** — `GET /me`, `PATCH /me`, `GET /me/card`; `GET /shooter/:membershipId` with rate limiting
7. **Login UI** — `Login.tsx` with email input; `/auth/verify` landing page that exchanges token and redirects to Dashboard
8. **Membership card** — `Card.tsx` with QR generation and print stylesheet

**Confirm Phase 1 complete before starting Phase 2.**

### Phase 2 — Core Scorecard

1. Session setup — `Setup.tsx`: org picker → round picker → grading toggle → club tag → shooter add
2. IndexedDB layer — `lib/db.ts` stores, `lib/sync.ts` Workbox queue
3. Session API — all session/shooter/score endpoints
4. `ScoreInput` component — config-driven zones, diminishing, stop-on-hit, timer
5. `Scoring.tsx` — target-by-target flow, back/next, post-target reveal
6. `Summary.tsx` — full scorecard, email trigger, save to history

---

## Hard Rules

- **Never hardcode discipline-specific logic in UI components.** All scoring behaviour comes from the discipline config object.
- **Grading toggle locks after first score.** `PATCH /sessions/:id` must reject changes to `is_grading` once `grading_locked = 1`.
- **Distances never shown during scoring** where `distance_visibility` is `after_target` or `never`. Non-negotiable rule compliance requirement.
- **v2 tables exist but are empty.** No API routes or UI for clubs, courses, or course_targets until v2 is explicitly started.
- **`session_shooters.shooting_style` is independent of profile preference.** Updating one never affects the other.
- **All API inputs validated with zod.** No unvalidated data reaches D1.
- **pnpm only.** Do not use npm or yarn.

---

## Environment Variables

Set via `wrangler secret put`. Store locally in `apps/worker/.dev.vars` (gitignored).

| Variable | Description |
|----------|-------------|
| `JWT_SECRET` | Random 32-byte hex string |
| `RESEND_API_KEY` | Resend API key |

---

## UI Style
All visual implementation must follow `STYLE-GUIDE.md` in this repository.

---

## When to Stop and Ask

Stop and ask before proceeding if you encounter:
- Any schema change not covered above
- Any new dependency not in the tech stack table
- Any deviation from the monorepo structure
- Any API route not in the endpoint table
- Any discipline scoring rule not covered here — the ABA National Rules (1 Jan 2026 v2) and IFAA Book of Rules (2021–2022) are available in the project context files

---

**Start with Phase 1, Task 1. Confirm completion before moving on.**
