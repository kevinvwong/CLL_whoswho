# Demo Seeding

## Purpose

Defines the scripted, idempotent demo-data pipeline: Pexels headshots treated consistently, fictional people with CLL-style team names, varied profile completeness, and a starter skill catalog.

## ADDED Requirements

### Requirement: Idempotent resettable seed
The seed process SHALL be runnable repeatedly: by default it must not duplicate existing seed data, and with an explicit reset option it must wipe and fully reseed. Seeding MUST leave the database in a state where the snapshot regenerates correctly.

#### Scenario: Double-run without duplication
- **WHEN** the seed runs twice without reset
- **THEN** the database contains no duplicated profiles or skills

#### Scenario: Reset reseeds cleanly
- **WHEN** the seed runs with the reset option
- **THEN** prior seed data is removed and the full demo population is recreated

### Requirement: Seeded photo treatment
The seed SHALL acquire portraits from the Pexels API at seed time only, download them, and produce bucketed thumbnail and profile sizes with a consistent crop aspect ratio and treatment across all profiles. Seeded photos MUST be marked photo-approved, and the runtime kiosk MUST NOT depend on Pexels CDN availability.

#### Scenario: Uniform treatment
- **WHEN** the seed completes
- **THEN** all profile photos share the same crop aspect and treatment

#### Scenario: Runtime independence from Pexels
- **WHEN** the kiosk renders seeded profiles
- **THEN** photo URLs resolve to the system's own storage, not Pexels

### Requirement: Varied completeness population
The seed SHALL generate a demo population (30+ profiles) with real CLL-style team names and fictional individuals, deliberately varied: some profiles fully populated across all tiers, some Tier-1-only, and at least one with a non-public visibility setting.

#### Scenario: Completeness variation present
- **WHEN** the seed completes
- **THEN** the population includes fully populated profiles, Tier-1-only profiles, and at least one profile with a suppressed field

### Requirement: Starter skill catalog
The seed SHALL populate a starter skill catalog (25+ skills across multiple categories) and associate most profiles with at least one skill, so the catalog dimension of the demo is populated.

#### Scenario: Catalog populated
- **WHEN** the seed completes
- **THEN** the skill catalog contains 25+ skills and most profiles have at least one skill association