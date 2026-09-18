# Profile Data (delta)

## ADDED Requirements

### Requirement: Person signals
A profile MAY carry optional signal fields adapted for a workplace context: `communication_style` (one line) and `strengths` (short tag list). Signals SHALL be treated as Tier-3 content: private by default, rendered in profile views when public, and never numeric or evaluative in tone.

#### Scenario: Signals render when public
- **WHEN** a profile has public communication-style and strengths signals
- **THEN** the profile view renders them in the culture section

#### Scenario: Signals default hidden
- **WHEN** a profile has signals but no visibility overrides
- **THEN** the signals do not appear on the public display

## MODIFIED Requirements

### Requirement: Skills association
A profile SHALL associate with zero or more catalog skills, each with an optional self-declared proficiency level (1–3). Skills exist in a shared catalog (name, category) so the same skill across profiles is one entity.

#### Scenario: Skill shared across profiles
- **WHEN** two profiles both declare the skill "Accessibility"
- **THEN** both reference the same catalog skill entry

#### Scenario: Profile with no skills is valid
- **WHEN** a profile has no skills associated
- **THEN** the profile remains valid and the skills section is omitted from its snapshot entry

### Requirement: Per-field visibility
Each Tier 2 and Tier 3 field SHALL carry a visibility setting of `public`, `staff`, or `private`. Fields default to `public` (Tier 2) or `private` (Tier 3). The kiosk display path MUST only receive data for fields marked `public`. Signal fields (`communication_style`, `strengths`) follow Tier-3 defaults (private).

#### Scenario: Visibility respected in snapshot
- **WHEN** the snapshot is generated and a profile's fun-fact field is set to `staff`
- **THEN** the snapshot omits the fun-fact for that profile

#### Scenario: Private default for human tier
- **WHEN** a profile is created with Tier 3 content but no visibility settings changed
- **THEN** the Tier 3 content is not exposed on the public display