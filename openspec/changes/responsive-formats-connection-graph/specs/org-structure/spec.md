# Org Structure (delta)

## MODIFIED Requirements

### Requirement: Org view on the kiosk
The kiosk SHALL offer an org-chart browse mode alongside the face grid: units ordered by hierarchy, each person tappable to their profile, with collaboration-link overlays visible from each org card (labeled, both internal and cross-team ties). The org view MUST obey the same touch and idle rules as the grid, and MUST reflow to the portrait format without losing hierarchy readability.

#### Scenario: Org browse mode
- **WHEN** a viewer selects the org view from the browse screen
- **THEN** the org chart renders with reporting structure visible and profiles remain one tap away

#### Scenario: Collaboration overlay
- **WHEN** an org card's person has collaboration ties
- **THEN** the card shows the linked people's names with labels

#### Scenario: Portrait org view
- **WHEN** the org view renders on a portrait kiosk
- **THEN** the hierarchy remains readable with narrower cards and preserved depth indentation

## ADDED Requirements

### Requirement: Connections browse integration
The connections browse mode (defined in connection-graph) MUST be reachable from the org view as well as the grid, and the org view's collaboration overlays MUST deep-link to the same profile views as the grid.

#### Scenario: Org to connections
- **WHEN** a viewer is in the org view
- **THEN** a connection-count summary per card links into the connections browse mode for that person's team