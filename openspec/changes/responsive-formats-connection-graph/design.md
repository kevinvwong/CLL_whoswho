# Design: Responsive Formats & Full Connection Graph

## Context

Builds directly on `kiosk-directory-mvp` (must archive first — this change's MODIFIED deltas target capabilities that MVP creates). Current state: Next.js 16 App Router app in `app/`, Drizzle + Neon/PGlite, snapshot-manifest display path, seeded 32-person demo with hand-written connections (works_with/mentors only, no strength), landscape-only kiosk UI with grid/profile/org views. BD_RPG (`C:\Users\kwong318\GitHub\BD_RPG`) provides the data-model reference: `bd_relationships` (typed, directional, labeled, sourced, confidence-scored relationships), `bd_interaction_edges` (edge_type + intensity 1–5), `bd_person_signals` (communication_style/personality_indicators/evidence), and its single-payload dashboard API pattern.

## Goals / Non-Goals

**Goals:**
- One codebase serving portrait kiosk, landscape kiosk, desktop browser (deep-linkable)
- Rich typed connection graph with strength, labels, and derived per-viewer rendering
- Person signals as a culture layer (workplace-appropriate adaptation of BD_RPG's signals)
- Connections browse mode (most-connected + team web lenses)

**Non-Goals:**
- No graph-visualization library (force-directed layouts etc.) — CSS/SVG only at this scale
- No editing surfaces (still deferred); no auth changes
- Not porting BD_RPG code — schema pattern inspiration only
- No mobile-phone-specific layout beyond "browser narrow = portrait layout"

## Decisions

### D1: Connection schema extension (additive)
Extend `connections`: add `strength integer null CHECK 1-5`, `note text null`. Keep `kind` as validated text (open vocabulary, app-side check against known kinds). Direction stays canonical in storage (mentor → mentee); rendering derives the viewer's perspective, as already implemented for `mentored_by`. Add `person_signals` as JSONB columns on `profiles` (communication_style, strengths) rather than a separate table — signals are 1:1 with profiles and follow the tier2/tier3 JSONB precedent. Alternative (full edges table à la bd_interaction_edges, per-episode rows) rejected: no temporal dimension in this domain; weight without provenance suffices for MVP+1.

### D2: Snapshot v2 payload (schema-versioned, additive)
```
{
  schemaVersion: 2,
  version, generatedAt,
  profiles: [{ ...existing, signals: {communicationStyle?, strengths?} }],
  roleSkills: [...],
  graph: {
    edges: [{ from, to, kind, label, strength }],   // canonical direction, deduped
    mostConnected: [{ slug, count }],                // precomputed, strength-weighted
    teamWeb: { [team]: [{ from, to, kind, label, cross: bool }] }
  }
}
```
`graph.mostConnected` and `graph.teamWeb` are precomputed at snapshot time so the kiosk never sorts at render. Snapshot reader (kiosk-data.ts) ignores unknown fields (schemaVersion guard satisfies the graceful-degradation scenario). Old kiosk clients keep working because all MVP fields are unchanged.

### D3: Responsive strategy (container/per-media queries, one component tree)
Same component tree, CSS-driven adaptation: `@media (orientation: portrait)` and a `~1024px` width breakpoint promote the portrait layout. Portrait specifics: grid columns `minmax(160px, 1fr)`, profile stacks (photo → content), attract mosaic uses 3-column rhythm, org tree compresses card padding but keeps depth indentation (48px → 32px). No separate portrait component tree — one tree, two skins. Deep links via a new `/p/[slug]` route that reuses ProfileView; kiosk state machine treats a deep-linked load as `state=profile` with `from="browser"` (idle timer disabled in browser context — detect via pointer:coarse media query + kiosk user-agent override env).

### D4: Connections browse mode (two lenses, precomputed in snapshot)
- Most-connected: rendered from `graph.mostConnected` — horizontal ranked cards with tie counts
- Team web: team picker chips → rendered from `graph.teamWeb[team]` — member tiles + tie list (cross-team ties marked)
Both lenses are list/card based, not node-link diagrams: at 32–100 people, node-link graphs on a touchscreen are unusable; ranked lists and team tables communicate density better and cost nothing to maintain. A future SVG force-graph is possible but out of scope.

### D5: Seed graph generation (algorithmic, not hand-written)
Replace the 14 hand-written CONNECTIONS with a generator: org-chart reporting lines stay hand-authored (they encode real structure), then the generator adds works_with ties within teams (2–4 per person, strength 1–4), cross-team project ties (labeled, from a project-name pool), mentor chains (2 hops deep), and advises ties from leads. Deterministic (seeded PRNG) so reseeds reproduce the same graph. Signals: 6 profiles get public communication_style + strengths from curated pools (workplace tone, per profile-data spec). Hub targets: ~3 people with 4+ ties, everyone ≥1 tie — asserted in seed verification.

### D6: BD_RPG borrow inventory (concrete)
| BD_RPG artifact | Borrow as |
|---|---|
| `bd_relationships` (person_a, person_b, rel_type, description, confidence) | connections table shape (kind, note; strength replaces intensity) |
| `bd_interaction_edges.edge_type` + `intensity` 1–5 | kind vocabulary + strength scale |
| `bd_person_signals` (communication_style, personality_indicators) | person signals JSONB (workplace-toned) |
| `bd_crew_photos` (person_id → img path) | existing photo URL convention (already aligned) |
| `/api/dashboard` single payload | snapshot manifest (already aligned) |
Explicitly not borrowed: CSV storage, Python engine, raycasting UI, episode/charter domain model, evidence/confidence fields (deferred — no research workflow here yet).

## Risks / Trade-offs

- [Snapshot growth from full graph] → Edges are small (~100 people × ~4 ties × ~100 bytes ≈ 40KB); 5s bound re-verified in tests.
- [Precomputed graph views go stale if connection data bypasses snapshot regeneration] → All writes flow through the same regenerate-on-write hook; seed verification asserts snapshot graph matches DB.
- [Portrait layout degrades org chart readability at depth 4+] → Cap visual indent scaling; test at 1080x1920 in tasks.
- [Open-kind vocabulary invites typos] → App-side validation warns on unknown kinds at write time; renderers group unknown kinds under "Other".
- [kiosk-directory-mvp still unarchived] → Its in-flight state (groups 4–5 nearly done) must complete and archive before this change's apply, or this change's MODIFIED deltas won't resolve. Sequencing note recorded in tasks.

## Migration Plan

1. Complete + archive `kiosk-directory-mvp` (its remaining UI/verification tasks are this repo's current apply state)
2. Apply this change: schema migration (additive) → snapshot v2 → seed regeneration → UI (responsive shell, deep links, connections browse) → E2E for new surfaces
3. Rollback: additive schema + additive payload mean the MVP kiosk keeps functioning against a v2 snapshot (unknown fields ignored); revert = deploy previous UI

## Open Questions

- Exact portrait kiosk hardware resolution — portrait layout is built against 1080x1920 and verified at 768x1366 as the small end; hardware specs still unknown (carried from MVP).
- Whether signals should ever render on the attract loop (rotating "people are interesting" moments) — polish decision, no spec dependency.