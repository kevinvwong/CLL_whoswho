# Demo Seeding (delta)

## MODIFIED Requirements

### Requirement: Varied completeness population
The seed SHALL generate a demo population (30+ profiles) with real CLL-style team names and fictional individuals, deliberately varied: some profiles fully populated across all tiers, some Tier-1-only, and at least one with a non-public visibility setting. The population MUST include a realistic rich connection graph: every profile in at least one connection, several hubs with 4+ ties, mixed kinds (works_with, mentors, collaborates_on, advises, project), mixed strengths (1–5), labeled project ties, and at least three profiles with public person signals (communication style + strengths).

#### Scenario: Completeness variation present
- **WHEN** the seed completes
- **THEN** the population includes fully populated profiles, Tier-1-only profiles, and at least one profile with a suppressed field

#### Scenario: Rich graph shape
- **WHEN** the seed completes
- **THEN** every profile has at least one connection, at least three profiles have four or more, and all relationship kinds from the vocabulary appear at least once

#### Scenario: Signals present
- **WHEN** the seed completes
- **THEN** at least three profiles carry public communication-style and strengths signals