# Directory Snapshot

## Purpose

Defines the single read-optimized JSON document the kiosk consumes: regenerated on any write, served as one endpoint, and containing only public display data.

## ADDED Requirements

### Requirement: Snapshot generation on write
The system SHALL regenerate the directory snapshot whenever any profile, skill, or visibility data is written. Snapshot generation MUST complete without partial states being served (readers never see a half-written snapshot).

#### Scenario: Write triggers fresh snapshot
- **WHEN** a profile's bio is updated
- **THEN** the served snapshot reflects the updated bio

#### Scenario: No torn reads
- **WHEN** a snapshot is being regenerated while a kiosk fetches it
- **THEN** the kiosk receives either the previous complete snapshot or the new complete snapshot, never a mix

### Requirement: Public-data-only snapshot
The snapshot MUST contain only data the public kiosk may show: Tier 1 fields, public-marked Tier 2/Tier 3 fields, approved photos, and catalog skill names. Staff-only and private field values MUST NOT appear anywhere in the snapshot document, including unused/empty field placeholders.

#### Scenario: Snapshot audit
- **WHEN** the snapshot JSON is inspected
- **THEN** no staff-only or private field values are present

### Requirement: Single-endpoint consumption
The kiosk SHALL fetch the entire directory dataset from one snapshot endpoint in a single request. The snapshot MUST include a generation timestamp and version the kiosk can use to detect staleness.

#### Scenario: One-request bootstrap
- **WHEN** the kiosk starts
- **THEN** it bootstraps all display data with a single snapshot request

### Requirement: Snapshot size bound
The snapshot for a fully seeded directory of at least 100 profiles with photos, skills, and Tier-3 content MUST be fetchable and parseable on the kiosk hardware within 5 seconds.

#### Scenario: Scaled snapshot performance
- **WHEN** 100+ profiles are seeded and the kiosk fetches the snapshot
- **THEN** the fetch-and-parse completes within 5 seconds