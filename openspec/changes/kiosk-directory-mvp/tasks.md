# Tasks: Kiosk Directory MVP

## 1. Scaffold & Infrastructure

- [x] 1.1 Scaffold Next.js App Router project with TypeScript; verify `npm run build` succeeds and dev server serves the root route
- [x] 1.2 Wire Drizzle + Neon HTTP driver with `isDatabaseConfigured()` guard (build passes without DATABASE_URL); verify build and a smoke query against a Neon branch URL
- [x] 1.3 Define Drizzle schema per design D5 (profiles, skills, profile_skills, field_visibility, snapshots) and verify `drizzle-kit generate` produces migrations cleanly

## 2. Seed Pipeline

- [x] 2.1 Implement Pexels fetch script (seed-time only): query portrait headshots, download originals; verify a test run stores N images locally with a Pexels key from env
- [x] 2.2 Implement image treatment: center-crop to unified aspect, generate thumb + profile buckets, upload to chosen storage; verify stored URLs resolve and sizes differ appropriately
- [x] 2.3 Implement demo population generator: 30+ fictional profiles with CLL-style team names, varied completeness (full / Tier-1-only / one suppressed field), fun facts, journeys; verify idempotent double-run adds no duplicates and `--reset` fully reseeds
- [x] 2.4 Seed starter skill catalog (25+ skills, multiple categories) and associate most profiles with ≥1 skill; verify counts via DB query
- [x] 2.5 Verify seeded photos are marked approved and all kiosk-facing photo URLs point at own storage, not Pexels (spec: demo-seeding)

## 3. Snapshot Manifest

- [ ] 3.1 Implement snapshot builder: public-data-only JSON assembly (visibility-filtered, approved photos only) with version + generated_at; verify snapshot audit test finds no staff/private values (spec: directory-snapshot)
- [ ] 3.2 Implement atomic snapshot swap (new row + `current` flip in one transaction) and a `current` reader; verify concurrent fetch during regeneration never sees a mix (test with interleaved reads)
- [ ] 3.3 Regenerate snapshot on every profile/skill write (seed + future edit hook); verify a bio update changes the served snapshot
- [ ] 3.4 Expose single GET snapshot endpoint; verify one-request bootstrap returns version, timestamp, and full payload; verify 100+ profile seed parses in <5s (measure in test)

## 4. Kiosk Display

- [ ] 4.1 Build brand token layer (CSS custom properties, placeholder GT palette, landscape-first container-query layout); verify no hardcoded colors outside tokens
- [ ] 4.2 Implement kiosk client data layer: fetch snapshot once at load, cache locally (IndexedDB), staleness-check interval, render-from-cache on fetch failure; verify offline tolerance by killing network mid-browse (spec: kiosk-display)
- [ ] 4.3 Build browse grid: scrollable face tiles (photo, name, title), ≥88px touch targets, one tap to profile; verify via Playwright touch interaction test
- [ ] 4.4 Build profile drill-in view: portrait, name, title, team, public Tier-2/3 fields, graceful Tier-1-only rendering, 24px-min body text; verify with a Tier-1-only seeded profile and a suppressed-field profile (spec: kiosk-display, profile-data)
- [ ] 4.5 Build idle attract loop: 90s no-touch timer resets any state to ambient drifting mosaic with rotating stats/quotes; touch wakes to grid; verify with an automated idle-timer test
- [ ] 4.6 Visual polish pass: transitions grid→profile (zoom), attract-loop drift motion, contrast against photo backgrounds; verify by screenshot review at 1920x1080 and a portrait fallback size

## 5. Verification & Demo Readiness

- [ ] 5.1 End-to-end demo flow test: fresh seed → snapshot serves → kiosk boots → attract → browse → profile → idle timeout (automated, headless)
- [ ] 5.2 Write README/demo runbook: env vars needed (DATABASE_URL, PEXELS_API_KEY), seed commands, run commands; verify a clean clone can reach the demo state following only the runbook