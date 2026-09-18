# Kiosk Display

## Purpose

Defines the public touchscreen experience: an ambient attract loop, touch-to-browse face grid, and drill-in profile view, tuned for standing viewers and self-resetting after use.

## ADDED Requirements

### Requirement: Idle attract loop
The kiosk SHALL display an ambient attract loop when no touch interaction has occurred for 90 seconds: a slow-drifting mosaic of profile portraits with periodic overlay stats or quotes. The attract loop MUST resume automatically from any browse or profile state after the idle timeout.

#### Scenario: Idle timeout returns to attract loop
- **WHEN** the screen receives no touch for 90 seconds while in browse or profile view
- **THEN** the kiosk transitions back to the attract loop

#### Scenario: Touch wakes the kiosk
- **WHEN** the kiosk is in the attract loop and a viewer touches the screen
- **THEN** the kiosk transitions to the face-grid browse view

### Requirement: Face-grid browsing
The kiosk SHALL present profiles as a scrollable grid of face tiles (photo, name, title), each tappable. Touch targets MUST be at least 88px. Browsing MUST require no more than one tap from the grid to a full profile.

#### Scenario: Tap a face tile
- **WHEN** a viewer taps a face tile in the grid
- **THEN** the full profile view for that person is displayed

### Requirement: Profile drill-in view
The kiosk SHALL display a full profile view showing, at minimum: portrait photo, name, title, team, and every Tier-2 and Tier-3 field the profile owner has marked public. Fields marked staff-only or private MUST NOT appear in the public kiosk view.

#### Scenario: Public fields render
- **WHEN** a profile's visibility flags mark its bio and skills public
- **THEN** the profile view shows the bio and skills

#### Scenario: Non-public fields suppressed
- **WHEN** a profile's visibility flags mark a field private or staff-only
- **THEN** that field does not render on the public kiosk

#### Scenario: Missing optional content
- **WHEN** a profile has no Tier-3 content filled in
- **THEN** the profile view renders the available Tier-1/Tier-2 content without errors or empty placeholders inviting input

### Requirement: Offline tolerance
The kiosk MUST continue rendering the most recently fetched snapshot data when the display API is unreachable. A connectivity failure MUST NOT blank the screen or show raw error text.

#### Scenario: Network drop during browsing
- **WHEN** the kiosk loses network connectivity while displaying browse or profile views
- **THEN** the current views continue to render from the last snapshot without error surfaces

### Requirement: Large-format legibility
The kiosk UI SHALL use a minimum body-text size of 24px-equivalent and high-contrast text against photo backgrounds, suitable for viewing from 1–2 meters.

#### Scenario: Profile body text size
- **WHEN** the profile view renders body content (bio, fun facts)
- **THEN** text renders at or above the 24px-equivalent minimum