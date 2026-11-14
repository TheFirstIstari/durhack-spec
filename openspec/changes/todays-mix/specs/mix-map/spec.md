# Spec Delta

## Purpose

Defines the map experience: a schematic-geographic, Factorio-style view of Britain's electricity composition with animated belts, three levels of granularity, drill-down to source, and art direction, performance and accessibility rules strict enough to test.

## ADDED Requirements

### Requirement: Schematic-geographic layout

The map SHALL place each region at its true geographic position using simplified shapes, and route belts along schematic corridors restricted to horizontal, vertical and 45-degree segments. Layout MUST NOT imply electrical topology the data does not establish.

#### Scenario: Region identification

- **WHEN** the national view loads
- **THEN** every distribution region is recognisable by position and shape, and every belt segment lies at 0, 45 or 90 degrees

### Requirement: Belts encode measured flow

Each interconnector SHALL render as a belt at its landing point whose direction, density and speed follow the signed measured flow, with items coloured by origin where known. Belt direction MUST remain legible with motion disabled.

#### Scenario: Importing and exporting links

- **WHEN** one interconnector is importing and another exporting
- **THEN** their belts move in opposite directions relative to the coast, with static direction markers visible even when animation is off

### Requirement: Region composition tiles

Each region SHALL display its generation composition as a stacked, fuel-coloured tile whose band heights follow the regional shares. Fuel colours MUST be identical on every view, legend and chart in the product.

#### Scenario: Windy north

- **WHEN** a region's wind share is above 80 percent
- **THEN** its tile is visibly dominated by the wind band, and its band proportions match the frame's shares

### Requirement: Provenance is visible

Provenance classes SHALL have distinct, documented visual encodings (for example solid for measured, hatched for upstream-modelled, outlined for derived, and a distinct frame treatment for scenario), explained by an always-reachable legend. Stale values MUST appear visibly degraded.

#### Scenario: Stale modality

- **WHEN** a modality becomes stale
- **THEN** its elements switch to the stale treatment and show their age on hover or focus

### Requirement: Drill-down to source

Selecting any region, link or asset SHALL reveal its values with unit, provenance class, source name, period, age and a path to the underlying raw record.

#### Scenario: Where does this number come from

- **WHEN** a user selects a region's wind share
- **THEN** the panel shows the value, its upstream-modelled class, the provider, the period covered and the age of the data

### Requirement: Three levels of granularity

The map SHALL provide national overview, region detail and unit/asset detail, revealing information through semantic zoom rather than scaling alone. Unplaced units MUST be accounted for in a visible count.

#### Scenario: Zoom into a region

- **WHEN** the user zooms into a region
- **THEN** additional detail appears (assets, units, weather), labels reflow, and belts and tiles keep correct values

### Requirement: Fuel composition by dragging

Users SHALL be able to drag fuel chips into and out of an active set to isolate or exclude fuels, with the map re-emphasising live as the chip moves. Only the settled set is written to the shareable URL.

#### Scenario: Remove gas from the picture

- **WHEN** the user drags the gas chip out of the active set
- **THEN** gas bands and items fade across all regions during the drag, and on drop the set is committed to the URL once

### Requirement: Documented art direction

The product SHALL ship a documented visual system: colour tokens for light and dark, a fuel palette validated for the three common colour-vision deficiencies, typography, belt and tile motifs, and motion rules. Text contrast MUST meet WCAG AA.

#### Scenario: Colour-vision check

- **WHEN** the fuel palette is simulated under protanopia, deuteranopia and tritanopia
- **THEN** every fuel remains distinguishable using colour plus a secondary cue (pattern, order or label)

### Requirement: Smooth interaction budget

During pan, zoom, scrub and drag, the map SHALL sustain a documented frame-rate target on a mid-range laptop, animate belts declaratively with no per-frame JavaScript, and cap concurrently animated elements at a documented limit. No server request is made per frame.

#### Scenario: Scrub while belts animate

- **WHEN** the user scrubs the timeline with all belts animating
- **THEN** measured frame time stays within budget and network inspection shows no per-frame requests

### Requirement: Reduced motion and assistive access

With a reduced-motion preference the map SHALL replace animation with static direction and magnitude cues and stay fully usable. All selectable elements MUST be keyboard reachable, and a data-table view of the current frame SHALL give screen-reader users the same information.

#### Scenario: Reduced motion enabled

- **WHEN** the system requests reduced motion
- **THEN** no belt animation runs, and direction and magnitude remain readable from static cues

#### Scenario: Keyboard-only user

- **WHEN** a user navigates with only a keyboard
- **THEN** they can select every region and link, open drill-down, and read the same values as the table view

### Requirement: Degraded and replay states are designed

Missing modality, failed integrity check, upstream outage and replay mode SHALL each have a distinct, designed treatment, and replay MUST be unmistakable on screen.

#### Scenario: Replay mode banner

- **WHEN** replay mode is active
- **THEN** a persistent replay marker is visible and the shown time is labelled as recorded, not live

### Requirement: Equivalent corridors show loading

The map SHALL draw the network model's equivalent corridors as belts between zones, classed derived in observed mode and scenario in scenario mode, with loading shown by colour plus a non-colour cue and overloads flagged. Corridors MUST be presented as equivalents of the transmission network, not physical lines, and MUST NOT use the measured encoding.

#### Scenario: Corridor belt present

- **WHEN** a corridor belt is drawn
- **THEN** it uses the derived or scenario encoding, shows utilisation numerically on focus, and its tooltip links to the method definition

#### Scenario: Overloaded corridor

- **WHEN** a corridor's loading exceeds its limit less the security margin
- **THEN** it shows the overload cue in addition to colour and is listed in the results panel

### Requirement: Published boundary values and confidence on the map

Where published day-ahead boundary values exist, the map SHALL show them against the model's value for the same period, label low-confidence boundaries, and show the residual on focus.

#### Scenario: Compare model and published

- **WHEN** a user focuses a mapped boundary
- **THEN** the modelled value, the day-ahead published value, the residual and the confidence class are shown together
