# Tasks: Responsive Formats & Full Connection Graph

## 0. Prerequisite

- [ ] 0.1 Confirm `kiosk-directory-mvp` is complete and archived (its MODIFIED delta targets require the base capabilities to exist in `openspec/specs/`); verify by `openspec list --specs` showing kiosk-display, profile-data, directory-snapshot, demo-seeding, org-structure

## 1. Connection Graph Data Layer

- [ ] 1.1 Extend schema per design D1: add `strength` (int, 1–5, nullable) and `note` (text, nullable) to connections; add `person_signals` JSONB (communication_style, strengths) to profiles; run `drizzle-kit generate` and verify the migration is additive; verify `npx tsc --noEmit` passes
- [ ] 1.2 Add app-side connection validation (kind vocabulary check, no self-connections, both profiles exist, directional kinds stored canonically) and verify rejected writes return errors (spec: connection-graph)
- [ ] 1.3 Build snapshot v2 payload per design D2: schemaVersion field, signals on profiles, `graph.edges` (canonical, deduped), precomputed `graph.mostConnected` (strength-weighted) and `graph.teamWeb` (cross-team marked); verify old fields unchanged and snapshot audit finds no private signals (spec: directory-snapshot)
- [ ] 1.4 Verify 100+ profile seeded snapshot with full graph fetches and parses in <5s (measure in test) (spec: directory-snapshot)

## 2. Seed: Rich Graph

- [ ] 2.1 Implement deterministic (seeded-PRNG) graph generator per design D5: within-team works_with ties (strength 1–4), cross-team labeled project ties, mentor chains, advises-from-leads; keep hand-authored reporting lines; verify every profile ≥1 tie, ≥3 hubs with 4+ ties, all vocabulary kinds present (spec: demo-seeding)
- [ ] 2.2 Seed signals for 6 profiles (public communication_style + strengths, workplace tone from curated pools); verify ≥3 public-signal profiles in snapshot (spec: demo-seeding, profile-data)
- [ ] 2.3 Verify seed idempotency and `--reset` still hold with the generator; verify snapshot graph matches DB rows exactly (spec: demo-seeding)

## 3. Responsive Display Shell

- [ ] 3.1 Refactor kiosk CSS per design D3: portrait layout via orientation media query + 1024px breakpoint (narrower grid columns, stacked profile, portrait attract rhythm, compressed org indentation); verify no data changes and landscape rendering unchanged (spec: responsive-display)
- [ ] 3.2 Verify live adaptation on window resize across orientation threshold (Playwright: set viewport 1920x1080 → 1080x1920, assert layout class change without reload) (spec: responsive-display)
- [ ] 3.3 Add mouse/keyboard affordances for browser format (hover states, wheel scroll, focus outlines); verify in desktop viewport with mouse interactions (spec: responsive-display)
- [ ] 3.4 Disable kiosk idle timer in browser context (pointer:coarse detection + env override); verify kiosk contexts still idle-return at 90s (spec: responsive-display, kiosk-display)

## 4. Deep Links

- [ ] 4.1 Add `/p/[slug]` route reusing ProfileView; cold-load renders loading state then profile; verify `/p/felix-zhang` direct navigation works with empty cache (spec: responsive-display)
- [ ] 4.2 Add not-found state for unknown slugs with a link back to the directory; verify `/p/nonexistent` (spec: responsive-display)

## 5. Connections Browse Mode

- [ ] 5.1 Build most-connected lens from `graph.mostConnected` (ranked cards, tie counts visible); reachable ≤2 taps from grid; each person one tap from profile (spec: connection-graph)
- [ ] 5.2 Build team-web lens: team picker chips → member tiles + tie list with cross-team marks; verify Media Production selection shows internal + cross-team ties (spec: connection-graph)
- [ ] 5.3 Add grid + org-view entry points to connections mode (one tap from grid per kiosk-display delta); verify org-card connection summaries deep-link correctly (spec: org-structure, kiosk-display)

## 6. Profile & Org Rendering Updates

- [ ] 6.1 Update profile drill-in: connections grouped by kind (strongest first, no numeric badges), reporting line ("Reports to X"), public signals in culture section; verify with a hub profile and a signals profile (spec: kiosk-display, connection-graph, profile-data)
- [ ] 6.2 Update org view: collaboration overlays labeled with names + project labels; portrait reflow keeps hierarchy readable; connection-count summary links into connections mode (spec: org-structure)
- [ ] 6.3 Verify directional rendering: mentor's profile shows "Mentors: X", mentee's shows "Mentored by: Y"; symmetric ties appear once per viewer without duplicates (spec: connection-graph)

## 7. Verification

- [ ] 7.1 Extend E2E suite: portrait viewport full flow (attract → grid → profile → org), deep-link cold load, connections browse both lenses, unknown-slug not-found; all headless-automated (spec: responsive-display, connection-graph)
- [ ] 7.2 Re-run existing E2E + idle tests to confirm no regression on landscape kiosk behavior; verify build passes with no DATABASE_URL (spec: kiosk-display)
- [ ] 7.3 Update README/demo runbook for new views, deep links, and portrait testing instructions; verify a clean clone can reach demo state following only the runbook