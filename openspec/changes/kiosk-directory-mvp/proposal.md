# Proposal: Kiosk Directory MVP ("Who's Who")

## Why

The CLL has no public-facing way to show who its people are, what they can do, or what makes the center's culture distinctive. A large public touchscreen in a common area is an opportunity to serve as a directory, a skill catalog, and a culture-building surface simultaneously — but only if the experience is visually excellent and trivially browsable by strangers. We need a demo-ready MVP to validate the concept with stakeholders before investing in real auth, real data import, and hardware tuning.

## What Changes

- New kiosk display application (read-only public surface): idle attract loop (ambient face mosaic + rotating stats/quotes), touch-to-browse face grid, and drill-in profile view.
- New companion data API: snapshot-manifest endpoint (single JSON document the kiosk consumes, offline-tolerant) backed by Neon Postgres via Drizzle.
- New data model: profiles (tiered: identity / professional / human), skill catalog with profile-skill associations, per-field visibility flags (public | staff | private), photo references (bucketed thumbnail + profile sizes stored as URLs; blobs in object storage).
- Seed pipeline for demo: scripted, idempotent, resettable; Pexels-sourced headshots (downloaded and bucketed at seed time with a consistent crop/treatment), fictional people with real CLL-style team names, deliberately varied profile completeness.
- Minimal dev-mode auth: reads fully public; edits open in dev (later swapped for SSO without schema change). Edit surfaces are deferred to a follow-up phase (P1) — this MVP is display + data foundation.
- Brand token layer with placeholder GT-style palette (e.g., `--cll-gold`) so visuals are brand-agnostic now and swap-in-ready when the real brand kit arrives.

## Capabilities

### New Capabilities
- `kiosk-display`: The public touchscreen experience — idle attract loop, face-grid browsing, profile drill-in, touch interaction rules (target sizes, state reset to idle, offline tolerance).
- `profile-data`: The profile data model and its semantics — tiered content, per-field visibility, photo bucketing, and the profile/skill association model.
- `directory-snapshot`: The snapshot manifest — how the display data is generated on write, served as a single document, and consumed offline-tolerantly by the kiosk.
- `demo-seeding`: The seed pipeline — idempotent demo-data generation, Pexels headshot acquisition and consistent image treatment, varied-completeness profiles, skill catalog starter set.

### Modified Capabilities

(none — greenfield)

## Impact

- **New codebase** in this repo (currently empty): Next.js app (kiosk display + API routes) with Drizzle ORM against Neon Postgres (`drizzle-orm/neon-http`).
- **New external dependencies**: Neon (hosted Postgres), Pexels API (seed-time only, key as env var, never a runtime dependency), an object-storage provider for photo blobs (URLs stored in Neon).
- **Deferred by design (P1+)**: self-serve profile editing, search/skill filtering, real SSO (Hexclave) swap, photo-approval workflow, HR/AD data import, kiosk hardware tuning (orientation, ambient light).
- **Out of scope**: real employee data (demo uses fictional people); production auth enforcement beyond read-publicity of the MVP scope.