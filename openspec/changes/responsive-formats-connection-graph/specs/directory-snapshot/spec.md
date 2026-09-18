# Directory Snapshot (delta)

## MODIFIED Requirements

### Requirement: Public-data-only snapshot
The snapshot MUST contain only data the public kiosk may show: Tier 1 fields, public-marked Tier 2/Tier 3 fields (including public-marked signals), approved photos, catalog skill names, the full connection graph (kind, direction-derived rendering info, label, strength), and role-skill mappings. Staff-only and private field values MUST NOT appear anywhere in the snapshot document, including unused/empty field placeholders.

#### Scenario: Snapshot audit
- **WHEN** the snapshot JSON is inspected
- **THEN** no staff-only or private field values are present, and no private-marked signals appear

### Requirement: Single-endpoint consumption
The kiosk SHALL fetch the entire directory dataset from one snapshot endpoint in a single request. The snapshot MUST include a generation timestamp and version the kiosk can use to detect staleness, and a schema-version field identifying the payload shape so older kiosk clients can degrade gracefully when fields are added.

#### Scenario: One-request bootstrap
- **WHEN** the kiosk starts
- **THEN** it bootstraps all display data with a single snapshot request

#### Scenario: Schema-version awareness
- **WHEN** a kiosk client built for an older payload shape fetches a newer snapshot
- **THEN** it renders known fields and ignores unknown ones without erroring

### Requirement: Snapshot size bound
The snapshot for a fully seeded directory of at least 100 profiles with photos, skills, Tier-3 content, the full connection graph, and signals MUST be fetchable and parseable on the kiosk hardware within 5 seconds.

#### Scenario: Scaled snapshot performance
- **WHEN** 100+ profiles with full graph data are seeded and the kiosk fetches the snapshot
- **THEN** the fetch-and-parse completes within 5 seconds