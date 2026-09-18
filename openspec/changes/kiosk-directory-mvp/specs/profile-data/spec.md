# Profile Data

## Purpose

Defines the person-record model: tiered content (identity, professional, human), per-field visibility control, photo bucketing, and profile-skill associations.

## ADDED Requirements

### Requirement: Tiered profile content
A profile SHALL consist of three tiers: Tier 1 identity (photo, name, title, team — required and public), Tier 2 professional (bio, skills, languages, "what I can help you with" — optional, public by default), and Tier 3 human (fun fact, journey, favorite tradition — optional, off by default). Tier 1 fields MUST NOT be individually hideable.

#### Scenario: Minimal profile is valid
- **WHEN** a profile is created with only Tier 1 fields populated
- **THEN** the profile is valid and renderable in the kiosk

### Requirement: Per-field visibility
Each Tier 2 and Tier 3 field SHALL carry a visibility setting of `public`, `staff`, or `private`. Fields default to `public` (Tier 2) or `private` (Tier 3). The kiosk display path MUST only receive data for fields marked `public`.

#### Scenario: Visibility respected in snapshot
- **WHEN** the snapshot is generated and a profile's fun-fact field is set to `staff`
- **THEN** the snapshot omits the fun-fact for that profile

#### Scenario: Private default for human tier
- **WHEN** a profile is created with Tier 3 content but no visibility settings changed
- **THEN** the Tier 3 content is not exposed on the public display

### Requirement: Photo bucketing
Each profile photo SHALL be stored as references to at least two pre-generated sizes: a thumbnail (for mosaic/grid tiles) and a profile image (for the drill-in view). Raw originals MUST NOT be referenced by the kiosk display path.

#### Scenario: Snapshot references bucketed sizes
- **WHEN** a profile has an approved photo
- **THEN** the snapshot includes thumbnail and profile-size URLs, and no raw-original URL

### Requirement: Skills association
A profile SHALL associate with zero or more catalog skills, each with an optional self-declared proficiency level (1–3). Skills exist in a shared catalog (name, category) so the same skill across profiles is one entity.

#### Scenario: Skill shared across profiles
- **WHEN** two profiles both declare the skill "Accessibility"
- **THEN** both reference the same catalog skill entry

#### Scenario: Profile with no skills is valid
- **WHEN** a profile has no skills associated
- **THEN** the profile remains valid and the skills section is omitted from its snapshot entry