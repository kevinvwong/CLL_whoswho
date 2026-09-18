# Connection Graph

## Purpose

Defines the full typed relationship model between people — kinds, direction, labels, strength, and evidence — and the graph-derived browse modes that make interconnection visible on the kiosk.

## ADDED Requirements

### Requirement: Typed relationships
Connections SHALL carry: a relationship `kind` from a fixed vocabulary (`works_with`, `mentors`, `collaborates_on`, `advises`, `project`), a directional flag (some kinds are directional: mentors, advises; others symmetric: works_with, collaborates_on), a human-readable optional `label` (project/event name), an optional strength 1–5, and an optional `note`. The vocabulary MUST be open to extension without schema change (kind is text, validated app-side).

#### Scenario: Directional relationship renders per viewer
- **WHEN** Hana mentors Lucas
- **THEN** Hana's profile shows "Mentors: Lucas" and Lucas's shows "Mentored by: Hana"

#### Scenario: Symmetric relationship renders once per viewer
- **WHEN** two people have a works_with tie
- **THEN** both profiles show the tie; no duplicates accumulate

#### Scenario: Strength ordering
- **WHEN** a profile lists connections
- **THEN** stronger connections (higher strength) sort before weaker ones within the same kind

### Requirement: Connection browse mode
The kiosk SHALL offer a connections browse mode alongside the face grid and org chart, with two lenses:
- **Most connected**: people ranked by total connection count (strength-weighted)
- **Team web**: for a chosen team, its members and their cross-team ties
Each lens MUST be reachable in at most two taps from the grid, and each listed person MUST be one tap from their profile.

#### Scenario: Most-connected lens
- **WHEN** a viewer opens the connections browse mode
- **THEN** the most-connected ranking renders with connection counts visible

#### Scenario: Team web lens
- **WHEN** a viewer picks the "Media Production" team
- **THEN** the view shows that team's members and their ties (internal and cross-team)

### Requirement: Graph integrity
Connection rows MUST reference existing profiles. Self-connections are invalid. Directional kinds MUST be stored in their canonical direction (mentor → mentee); the display layer derives the per-viewer rendering.

#### Scenario: Invalid connection rejected
- **WHEN** a write creates a connection referencing a nonexistent profile or self
- **THEN** the write is rejected

### Requirement: Strength and notes surface minimally
Strength SHALL render as visual weight (chip ordering, subtle emphasis), never as a numeric badge on the kiosk. Notes SHALL render as tooltip/subtext in browser format and MAY be omitted on the kiosk profile view when space-constrained.

#### Scenario: Kiosk rendering
- **WHEN** a profile has 8 connections with mixed strengths
- **THEN** the kiosk renders them grouped by kind, strongest first, without numeric indicators