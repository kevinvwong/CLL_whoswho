# Org Structure

## Purpose

Defines the organizational ontology: how people, roles, and org units interconnect — reporting lines for the org chart, role-to-skill mappings, and collaboration/mentorship ties that form the full connection graph.

## ADDED Requirements

### Requirement: Reporting lines
A profile MAY declare a `reports_to` reference to another profile (or none, for heads). The org chart MUST be derivable from this field alone, and the data MUST remain a forest (cycles are invalid).

#### Scenario: Org chart from reporting lines
- **WHEN** profiles declare reports_to relationships
- **THEN** the org chart renders as a tree from the head(s) down

#### Scenario: Cycle rejected
- **WHEN** a write would create a cycle in reporting lines
- **THEN** the write is rejected

### Requirement: Connection graph
Profiles MAY be connected by typed relationships: `collaborates_with` (project ties) and `mentors` (directional mentorship). Each connection carries an optional label (e.g., a project name). Connections MUST be queryable per profile and MUST render on the kiosk profile view.

#### Scenario: Profile shows connections
- **WHEN** a profile has collaboration or mentorship ties
- **THEN** its kiosk profile view lists the connected people by name with the relationship type and label

#### Scenario: Bidirectional collaboration
- **WHEN** profile A collaborates with profile B
- **THEN** both A and B see the connection on their profiles

### Requirement: Role-skill mapping
Each distinct title SHALL map to a role entity that declares the skills that role enables or requires. The kiosk MUST be able to show, for any skill, which roles (and people) cover it, and for any role, its required skills.

#### Scenario: Skill to role lookup
- **WHEN** a viewer selects a skill
- **THEN** the kiosk can show the roles that include it and the people holding those roles

### Requirement: Org view on the kiosk
The kiosk SHALL offer an org-chart browse mode alongside the face grid: units ordered by hierarchy, each person tappable to their profile, with connection links visible from each profile. The org view MUST obey the same touch and idle rules as the grid.

#### Scenario: Org browse mode
- **WHEN** a viewer selects the org view from the browse screen
- **THEN** the org chart renders with reporting structure visible and profiles remain one tap away

### Requirement: Org data in the snapshot
The directory snapshot SHALL include reporting lines (by profile slug), typed connections (by slugs, with labels), and role-skill mappings so the kiosk renders the org view and profile connections offline from the same single request.

#### Scenario: Snapshot carries org data
- **WHEN** the snapshot is fetched
- **THEN** each profile entry includes its reportsTo slug (or null), its typed connections, and role-skill mappings are present