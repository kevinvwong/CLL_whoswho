# Proposal: Responsive Formats & Full Connection Graph ("as envisioned")

## Why

The MVP kiosk (change `kiosk-directory-mvp`) proves the concept on a single landscape touchscreen. The full vision is broader: the same profile and connection data must serve **multiple formats** — portrait kiosks, desktop/laptop browsers, and phones — each with an experience tuned to its context. And the connection model seeded from the MVP (a handful of hand-written ties) must grow into a **rich, curated connection graph** in the spirit of the BD_RPG analytics dataset: typed, labeled, directional relationships with evidence, plus per-person communication-style signals — so the kiosk answers "how do these people and roles interconnect?" not just "who exists."

## What Changes

- **Multi-format display**: the kiosk UI becomes fully responsive with two first-class formats: a portrait layout for portrait-oriented kiosks (taller grid, stacked profile, portrait-tuned attract loop) and the existing landscape layout; plus a **browser format** for standard desktop browsers (mouse-first, resizable windows, deep-linkable profile URLs like `/p/<slug>` so profiles can be shared via email/QR).
- **Full connection graph**: connections expand to a typed relationship model inspired by BD_RPG's `bd_relationships`/`bd_interaction_edges` schema — relationship kinds (works-with, mentors/mentored-by, collaborates-on, advises, allied-project), directional semantics, optional labels, optional strength/intensity, and per-profile connection summaries. The snapshot carries the whole graph; the kiosk org view overlays collaboration links; profile views render grouped connection lists.
- **Person signals (culture layer)**: BD_RPG's `bd_person_signals` pattern adapted — optional per-person "communication style" / "strengths" tags that surface in profiles, seeded for demo people.
- **Graph API & future-proofing**: the snapshot schema versioned; a graph-derived "connections browse" mode (browse by relationship density: most-connected people, per-team web) added as a kiosk view.

### Relationship to BD_RPG (what we borrow vs. don't)

**Borrowed (patterns + schema inspiration):**
- Typed relationship rows with `kind`, direction, `label`, `source`, `confidence` — directly from BD_RPG's `bd_relationships` CSV shape
- Per-episode/per-project interaction edges with `edge_type` + `intensity` (1–5) — from `bd_interaction_edges`
- Person signal rows (communication_style, personality_indicators, evidence) — from `bd_person_signals`
- The "single JSON payload" dashboard API pattern (BD_RPG's `/api/dashboard`) — mirrors our snapshot manifest
- The photo-by-person-id convention (`bd_crew_photos`)

**Not borrowed:** BD_RPG's Python engine, raycasting UI, CSV-based storage, episode/charter domain tables, or its Vercel/stdlib-server deployment. This project remains Next.js + Drizzle + Neon; BD_RPG is read as a **data-model and content-curation reference only**.

## Capabilities

### New Capabilities
- `responsive-display`: Multi-format display rules — portrait kiosk, landscape kiosk, desktop browser, deep-linkable profile URLs, and format-specific interaction tuning.
- `connection-graph`: The full typed relationship model — kinds, direction, labels, strength, per-profile summaries, and graph-derived browse modes.

### Modified Capabilities
- `profile-data`: connections become the typed graph entries (kind/direction/label/strength) instead of the MVP's flat list; person-signal fields added as an optional Tier-3 extension.
- `directory-snapshot`: snapshot payload extended with full graph data and person signals (still public-data-only, still one request).
- `org-structure`: org view gains collaboration-link overlays and strength-aware rendering; role-skill lookup now powers a connections browse mode.
- `demo-seeding`: seed generates a realistic rich graph (varied kinds, strengths, labels, a few signals) rather than a handful of hand-written ties.
- `kiosk-display`: adds responsive breakpoints, deep-link routing, and the connections browse mode to the display contract.

## Impact

- **Code (app/)**: kiosk.tsx + kiosk.module.css (responsive layouts, connections browse view, deep links); snapshot.ts (payload extension); schema.ts (connection columns: strength, source; person_signals table); seed scripts (rich graph generation); new `/p/[slug]` route for browser deep links.
- **New dependencies**: none required (graph rendering stays CSS/SVG; no graph library for MVP scale).
- **Data**: snapshot JSON grows (~2–3x with full graph); the 5s/100-profile bound must be re-verified.
- **No breaking changes** to the existing kiosk contract — portrait and browser formats extend it; snapshot remains additive (new optional fields).
- **Deferred**: real auth, edit surfaces, HR import (unchanged from MVP deferrals).