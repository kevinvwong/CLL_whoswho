# Kiosk Display (delta)

## MODIFIED Requirements

### Requirement: Face-grid browsing
The kiosk SHALL present profiles as a scrollable grid of face tiles (photo, name, title), each tappable. Touch targets MUST be at least 88px. Browsing MUST require no more than one tap from the grid to a full profile. The grid MUST include persistent entry points to the org chart and connections browse modes, each at most one tap away.

#### Scenario: Tap a face tile
- **WHEN** a viewer taps a face tile in the grid
- **THEN** the full profile view for that person is displayed

#### Scenario: Entry points to other views
- **WHEN** the grid renders
- **THEN** org-chart and connections browse controls are visible and reachable in one tap

### Requirement: Profile drill-in view
The kiosk SHALL display a full profile view showing, at minimum: portrait photo, name, title, team, reporting line ("Reports to X" when present), every Tier-2 and Tier-3 field the profile owner has marked public (including public signals), and their connections grouped by kind (strongest first). Fields marked staff-only or private MUST NOT appear in the public kiosk view.

#### Scenario: Public fields render
- **WHEN** a profile's visibility flags mark its bio and skills public
- **THEN** the profile view shows the bio and skills

#### Scenario: Connections grouped by kind
- **WHEN** a profile has works_with, mentors, and project connections
- **THEN** the profile view groups them under labeled kind headings, strongest first

#### Scenario: Reporting line shows
- **WHEN** a profile has a reports_to relationship
- **THEN** the profile view shows who they report to

#### Scenario: Non-public fields suppressed
- **WHEN** a profile's visibility flags mark a field private or staff-only
- **THEN** that field does not render on the public kiosk

#### Scenario: Missing optional content
- **WHEN** a profile has no Tier-3 content filled in
- **THEN** the profile view renders the available Tier-1/Tier-2 content without errors or empty placeholders inviting input

## ADDED Requirements

### Requirement: Format-aware display shell
The kiosk display shell SHALL select portrait or landscape layouts per responsive-display, and support the browser deep-link routes without changing kiosk behavior. All existing kiosk rules (idle, touch size, offline) hold in both orientations.

#### Scenario: Deep link renders profile
- **WHEN** `/p/<slug>` is opened on any format
- **THEN** the profile view renders directly with a path back to the directory