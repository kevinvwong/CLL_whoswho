# CLL "Who's Who" — Complete Project Specification

**Version:** 1.0 (2026-09-17)
**Status:** Envisioned system — synthesizes the `kiosk-directory-mvp` (in flight) and `responsive-formats-connection-graph` (planned) changes plus the full product vision discussed in explore mode.

---

## 1. Purpose

The CLL operates a large public touchscreen in a common area. This system turns that screen — and any browser — into three products wearing one interface:

1. **Directory** — who is at the CLL, what do they do, how do I find them?
2. **Skill catalog** — who can help me with X? What capabilities live in this center?
3. **Culture building** — who are these people as humans? What journeys, traditions, and connections make the CLL what it is?

The screen is public; the data is personal. Every person controls, field by field, what the world sees.

---

## 2. Product Surfaces

```
+---------------------+     +--------------------+     +--------------------+
|  Touchscreen kiosk  |     |  Companion browser |     |  Admin / seeding   |
|  (portrait or       |     |  (self-serve CRUD, |     |  (photo approval,  |
|   landscape,        |     |   deep links,      |     |   HR import —      |
|   read-only,        |     |   mouse/keyboard)  |     |   future phase)    |
|   ambient + touch)  |     |                    |     |                    |
+----------+----------+     +---------+----------+     +---------+----------+
           |                          |                          |
           +--------------------------+--------------------------+
                                      |  one API, one source of truth
                                      v
                          +------------------------+
                          |   Next.js app          |
                          |   snapshot manifest    |
                          +-----------+------------+
                                      |  Drizzle (HTTP driver)
                                      v
                          +------------------------+
                          |      Neon Postgres     |
                          +------------------------+
                                      |
                          photo blobs: static buckets
                          (thumb + profile sizes)
```

### 2.1 Kiosk (touch, public)

- **Idle attract loop** — ambient drifting portrait mosaic, rotating stats/quotes ("127 people. One mission."), pulsing "Touch to explore." Auto-resumes after 90s of no touch from any state.
- **Face grid** — scrollable tiles (photo, name, title), ≥88px touch targets, one tap to profile. Entry points to org chart and connections views.
- **Profile drill-in** — portrait, name, title, team, reporting line, public Tier-2/3 content, skills grouped by category (★ = expert), connections grouped by kind (strongest first), person signals.
- **Org chart view** — reporting hierarchy as an indented tree; collaboration overlays on cards ("⋯ Maya Okafor (Annual Demo Day)"); portrait reflow preserves readability.
- **Connections browse** — two lenses: *most connected* (strength-weighted ranking) and *team web* (member tiles + cross-team ties). List/card based, not node-link graphs.
- **Offline tolerance** — renders from cached snapshot when network drops; never blanks, never shows raw errors.
- **Formats** — landscape (1920×1080-class) and portrait (1080×1920) layouts from one component tree; live adaptation on rotation/resize; body text ≥24px-equivalent.

### 2.2 Browser (mouse/keyboard, semi-public)

- Same data, tuned interaction: hover states, wheel scroll, focus outlines.
- **Deep-linkable profiles**: `/p/<slug>` cold-loads any profile; unknown slugs get a friendly not-found.
- Idle timeout relaxed (browser isn't a kiosk).
- Narrow windows get the portrait layout.

### 2.3 Editing (future phase)

Self-serve CRUD via a companion web app ("act as X" stub in dev, SSO later). Per-field visibility toggles are the quiet hero: an employee decides, field by field, public vs. staff-only vs. private — which is what makes people willing to fill in the human layer.

---

## 3. Data Model

### 3.1 Profiles (tiered content)

| Tier | Fields | Default visibility |
|---|---|---|
| **1 — Identity** (required, always public) | photo (bucketed thumb+profile), name, title, team, `reports_to` | public |
| **2 — Professional** | bio, skills[] (level 1–3), languages, "what I can help you with" | public |
| **3 — Human** | fun fact, journey, favorite tradition, **signals**: communication style, strengths | **private** |

Per-field visibility: `public | staff | private`. The display path only ever receives `public` data — enforced by construction in the snapshot, not by client filtering.

### 3.2 Connection graph (BD_RPG-inspired)

Typed relationships between people:

```
connections: from_profile -> to_profile
  kind:     works_with | mentors | collaborates_on | advises | project   (open vocabulary)
  label:    optional (project/event name, e.g. "Annual Demo Day")
  strength: 1–5, optional (visual weight only — never a numeric badge)
  note:     optional subtext (browser tooltip; omitted on kiosk when tight)
```

- **Directional kinds** (mentors, advises) stored canonically (mentor → mentee); rendering derives the viewer's perspective ("Mentors X" / "Mentored by Y").
- **Symmetric kinds** (works_with, collaborates_on) render once per viewer.
- Integrity: no self-connections, both endpoints must exist.
- Borrowed from BD_RPG's `bd_relationships` / `bd_interaction_edges` (typed, labeled, intensity-scaled relationship rows). Evidence/confidence fields are deferred until a research workflow exists.

### 3.3 Role ontology

- Each distinct title maps to a **role** with the skills it enables → "who covers skill X?" lookups.
- `reports_to` forms the org forest (cycles invalid) → the org chart is derivable from this field alone.

### 3.4 Person signals (culture layer)

Optional, workplace-toned: `communication_style` (one line) and `strengths` (short tags). Tier-3 defaults (private). Inspired by BD_RPG's `bd_person_signals`, minus the evaluative/research framing.

### 3.5 Skills catalog

Shared catalog (name, category — 28 seeded across 6 categories); many-to-many with profiles via `profile_skills` (optional proficiency level).

---

## 4. Architecture

- **Stack**: Next.js 16 App Router (one app: display + API), TypeScript, Drizzle ORM, Neon Postgres (HTTP driver; PGlite for local dev), Tailwind-free CSS module system with brand token layer.
- **Snapshot manifest** (the core display pattern): every write regenerates a single JSON document — atomic swap (new row + `current` pointer flip in one transaction, no torn reads). The kiosk bootstraps with **one request**, caches in IndexedDB, polls for staleness, renders from cache when offline.
- **Public-data-only by construction**: staff/private values never enter the snapshot.
- **Snapshot v2 payload** (planned): `schemaVersion` for graceful degradation; precomputed `graph.edges`, `graph.mostConnected` (strength-weighted), `graph.teamWeb`.
- **Photos**: Pexels at seed time only → unified 4:5 crop (background-uniformity-scored selection, face-tight framing) → own static storage (`/photos/{thumb,profile}`) → URLs in DB. Runtime never depends on Pexels.
- **Dev auth**: none (edit routes env-flag gated). Real SSO is a later swap, not a migration.
- **Brand**: placeholder GT-plausible token layer (`--cll-gold` etc.); real brand kit swaps tokens, not components.

### 4.1 What BD_RPG contributes (pattern reference only)

| BD_RPG artifact | Borrowed as |
|---|---|
| `bd_relationships` (typed, labeled, directional) | connections table shape |
| `bd_interaction_edges.edge_type` + `intensity` 1–5 | kind vocabulary + strength scale |
| `bd_person_signals` | person signals (workplace tone) |
| `bd_crew_photos` (id → img path) | photo URL convention |
| `/api/dashboard` single payload | snapshot manifest pattern |

Not borrowed: Python engine, CSV storage, raycasting UI, episode/charter domain, evidence/confidence workflow.

---

## 5. Seeding & Demo

- 30+ fictional people, real CLL-style team names, deliberately varied completeness (full / Tier-1-only / suppressed field) — empty states are a feature (incentive to fill), rendered gracefully.
- Deterministic graph generator (seeded PRNG): every person ≥1 tie, ~3 hubs with 4+ ties, all relationship kinds present, labeled cross-team project ties, mentor chains.
- 6 profiles with public signals. ≥3 asserted.
- Idempotent; `--reset` wipes and reseeds; snapshot regenerates on every seed/write.
- Seed doubles as the schema's first integration test.

---

## 6. Phasing

| Phase | Contents | Status |
|---|---|---|
| **P0 — MVP** | Schema+DB, Pexels seed, snapshot manifest, landscape kiosk (grid/profile/attract), E2E | ✅ ~90% (groups 4–5 finishing) |
| **P1 — This vision** | Full connection graph (strength/labels/signals), responsive portrait + browser formats, deep links, connections browse | Planned (`responsive-formats-connection-graph`) |
| **P2 — Real operations** | SSO (Hexclave), self-serve editing, photo approval workflow, HR/AD import, per-field visibility UI | Deferred |
| **P3 — Culture amplification** | Attract-loop human moments, SVG force-graph (if scale demands), evidence/provenance for connections, analytics | Future |

---

## 7. Non-Goals

- No real employee data in demos (fictional people, real structure).
- No graph-visualization libraries at this scale (lists/cards beat node-link on touchscreens).
- No evaluation scores, rankings, or metrics on people — strength is visual weight, never a number.
- No mobile-phone-specific layout beyond "narrow browser = portrait layout."
- No editing on the kiosk itself.

## 8. Success Criteria (demo-ready bar)

- A stranger standing at the screen: attracted → browsing a profile in under 10 seconds, unassisted.
- The connection graph visibly answers "how do these people interconnect?" — org lines, mentorship, projects.
- Reseed-to-demo in one command; snapshot serves the whole directory in one request, <5s at 100+ profiles.
- Same data, three formats, zero data drift.