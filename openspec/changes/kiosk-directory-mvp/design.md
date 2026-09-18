# Design: Kiosk Directory MVP

## Context

Empty greenfield repo. The system is three surfaces sharing one data model: a public kiosk (read-only, offline-tolerant), a future companion web app (self-serve CRUD, P1), and a future admin surface. For the MVP only the kiosk display and its data foundation are built. DB is Neon Postgres; minimal dev auth; demo data seeded via Pexels. Brand kit not yet supplied — placeholder tokens required. See proposal.md for motivation and specs/ for behavior contracts.

## Goals / Non-Goals

**Goals:**
- Demo-ready kiosk experience: attract loop → browse grid → profile drill-in, visually polished
- Data foundation that the P1 edit surface can adopt without schema change
- Offline-tolerant display path (snapshot manifest pattern)
- One-command demo reset for stakeholder iterations

**Non-Goals:**
- Real auth (SSO swap later; dev edits are open/stubbed)
- Search/skill filtering UI (P1), photo-approval workflow (P2), HR import (P2)
- Real employee data of any kind

## Decisions

### D1: Next.js App Router, one app for display + API
One Next.js project: kiosk display route, API routes for snapshot. Alternative (separate kiosk SPA + backend) rejected — two deploy targets for one demo; Next.js server components suit large-media rendering and we keep the option to add the companion edit app as routes in the same repo later.

### D2: Neon via `drizzle-orm/neon-http`
HTTP driver per query; no persistent connection needed. Fits serverless API routes and the known-good pattern (pure `isDatabaseConfigured()` guard so builds pass without a real URL). Neon branching gives per-preview-throwaway DBs for testing seeds.

### D3: Snapshot manifest as the display data path
Any write regenerates a single JSON document (`directory snapshot`) stored in the DB (a `snapshots` table holding the JSON + version + timestamp); the kiosk fetches it with one request and caches locally (IndexedDB) for offline tolerance. Atomic swap: write new row, flip a `current` pointer in one transaction — no torn reads. Alternative (edge-cached live API) rejected: offline tolerance and simplicity beat freshness; profiles change weekly at most. Snapshot is public-data-only by construction (see profile-data spec).

### D4: Photos in object storage, URLs in Neon; Pexels is seed-time only
Buckets: `thumb` (grid) and `profile` (drill-in), generated at seed time with a unified center-crop + treatment. Originals never exposed to the kiosk. Choice of blob provider (Vercel Blob vs. Neon-adjacent static storage) is an implementation detail — store only opaque URLs in the DB. Pexels key is a build/seed env var, never runtime.

### D5: Schema (Drizzle, single Neon project)
```
profiles(id, slug, name, title, team, tier2 jsonb, tier3 jsonb,
         photo_thumb_url, photo_profile_url, photo_approved,
         created_at, updated_at)
skills(id, name unique, category)
profile_skills(profile_id, skill_id, level 1-3 nullable)
field_visibility(profile_id, field_key, visibility)   -- overrides defaults
snapshots(id, version, generated_at, payload jsonb, current bool)
```
Tier 2/3 as JSONB keeps the per-field visibility mapping simple (field_key → visibility) and avoids column churn as fields evolve. Relational tables only where we query across profiles (skills, photos, snapshot).

### D6: Dev auth = none, edit routes gated by env flag
Edits are dev-only and behind an env flag (`EDIT_MODE=dev`). Reads are public by design. Schema carries no auth columns; SSO (Hexclave) is a P2 addition, not a migration.

### D7: Brand token layer
CSS custom properties (`--cll-primary`, `--cll-gold`, etc.) with placeholder GT-plausible values; all styling references tokens only. Real brand kit swaps tokens, not components. Portrait orientation unknown — layout uses container queries and works landscape-first; orientation flip is a token/layout tweak.

### D8: Kiosk client architecture
Single-page client view: attract loop and browse grid as states of one component tree, transitions via view-transitions/Framer-free CSS. State machine: `idle → grid → profile`, any state → `idle` on 90s no-touch timer. Fetches snapshot on load + on interval (staleness check); renders from local cache when fetch fails.

## Risks / Trade-offs

- [Pexels photo style variance undermines "beautiful"] → Unified crop/treatment in seed; manual curation pass over the seeded set is acceptable for a demo.
- [Snapshot size growth with Tier-3 richness] → 100-profile bound in spec; if exceeded, move Tier-3 to lazy per-profile fetch (schema unchanged).
- [Fictional people mistaken for real] → Seed names drawn from clearly synthetic name list; demo header labels "Demo data" unobtrusively during dev only.
- [Public screen shows incomplete profiles that look broken] → Profile view designed to render gracefully from Tier-1-only (spec'd); completeness variation in seed makes this a feature (incentive to fill), not a bug.
- [Touchscreen hardware unknown (orientation, resolution, brightness)] → Landscape-first + container queries; hardware tuning task exists in P2.

## Migration Plan

Greenfield: scaffold app → schema + Neon wiring → seed pipeline → snapshot generation → display UI. Rollback is trivial pre-launch (no users). Neon branch used for seed-iteration work; main branch promoted once seed stabilizes.

## Open Questions

- Exact blob-storage provider for photo buckets (Vercel Blob vs. plain static hosting) — decidable at implementation time without spec impact.
- Attract-loop content rotation cadence and stat/quote sourcing — polish detail, no spec dependency.