# Responsive Display

## Purpose

Defines how the directory serves multiple display formats — portrait kiosk, landscape kiosk, and desktop browser — each with tuned layout and interaction, and how profiles are deep-linkable in browser contexts.

## ADDED Requirements

### Requirement: Format detection and layout selection
The display app SHALL render a portrait-optimized layout when the viewport is taller than wide, and the landscape layout otherwise. The portrait layout MUST reflow the face grid (narrower columns, taller scroll), stack the profile view (photo above content), and adapt the attract loop (portrait mosaic rhythm) — all without data changes.

#### Scenario: Portrait kiosk layout
- **WHEN** the display loads on a portrait-orientation screen (e.g., 1080x1920)
- **THEN** the face grid renders narrower columns and the profile view stacks photo above content

#### Scenario: Landscape unchanged
- **WHEN** the display loads on a landscape screen
- **THEN** the existing landscape layout renders

#### Scenario: Rotation adapts live
- **WHEN** a browser window is resized across the orientation threshold
- **THEN** the layout adapts without reload

### Requirement: Browser format
The app SHALL render a desktop-browser-tuned experience when interacted with via mouse (hover affordances, scroll wheel navigation, keyboard focus states). Browser windows narrower than kiosk scale MUST render the portrait layout scaled to window size.

#### Scenario: Mouse browsing
- **WHEN** a desktop user browses with a mouse
- **THEN** hover states highlight tiles and scrolling works via wheel and scrollbar

### Requirement: Deep-linkable profiles
The browser format SHALL support direct profile URLs (`/p/<slug>`) that render the profile view immediately. Deep links MUST work from a cold load (no prior snapshot in cache), falling back to the loading state until data arrives. Unknown slugs MUST show a not-found state, never a crash.

#### Scenario: Shared profile link
- **WHEN** a user opens `/p/felix-zhang` directly in a browser
- **THEN** Felix Zhang's profile renders

#### Scenario: Unknown slug
- **WHEN** a user opens `/p/nonexistent`
- **THEN** a not-found state renders with a link back to the directory

### Requirement: Kiosk formats preserve kiosk rules
All formats on kiosk hardware MUST retain the idle timeout, touch-target minimums, and offline tolerance from kiosk-display. Browser format (non-kiosk) MAY relax the idle timeout.

#### Scenario: Portrait kiosk idles
- **WHEN** a portrait kiosk receives no touch for 90 seconds
- **THEN** it returns to the attract loop