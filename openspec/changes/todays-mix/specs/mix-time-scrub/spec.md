# Spec Delta

## Purpose

Defines how users move through time: following the live head, scrubbing and playing back recent history, and reading time unambiguously across UTC, local time and settlement periods, including clock-change days.

## ADDED Requirements

### Requirement: Live head with a local history window

The map SHALL follow the latest frame by default and hold a history window of at least 48 hours in the browser, so scrubbing never waits on the network.

#### Scenario: Open the product

- **WHEN** a user opens the map
- **THEN** it shows the latest frame as live and the full history window is available to scrub after the initial load

### Requirement: Scrub by dragging, with no per-frame requests

The timeline handle SHALL be draggable, update the map continuously while dragging from locally held frames, and make no server request per drag frame. Only the settled time is committed to the shareable URL, once per drop.

#### Scenario: Drag across a day

- **WHEN** the user drags the handle across 24 hours of history
- **THEN** the map updates continuously with no network requests during the drag, and one URL update on drop

### Requirement: Scrubber track shows composition over time

The timeline track SHALL render the national composition across the window as a fuel-coloured stacked stream, so the shape of the day is visible before scrubbing.

#### Scenario: Reading the day at a glance

- **WHEN** the history window is loaded
- **THEN** the track displays each fuel's share over time using the product's fuel colours, aligned to the handle position

### Requirement: Playback

Users SHALL be able to play the history window forward at selectable speeds, pause, and step between frames.

#### Scenario: Play a day

- **WHEN** the user starts playback at a faster speed
- **THEN** the map advances through frames at that speed and stops at the live head

### Requirement: Live following does not yank the user

When a new frame arrives and the user is at the live head, the map SHALL advance smoothly. If the user has scrubbed into the past, the view MUST stay put, and a clear control SHALL return them to live.

#### Scenario: New frame while browsing history

- **WHEN** a new frame is ingested while the user is viewing an earlier time
- **THEN** their view does not change, the timeline extends, and a return-to-live control is shown

### Requirement: Unambiguous time

All timestamps SHALL be held in UTC and displayed in UK local time with the settlement period where relevant. Clock-change days, including the 46 and 50 settlement-period days, MUST be displayed without duplicated, skipped or mislabelled periods.

#### Scenario: Autumn clock change

- **WHEN** the window spans the end of British Summer Time
- **THEN** the repeated local hour is shown distinctly, both occurrences map to correct frames, and the day has 50 settlement periods

#### Scenario: Spring clock change

- **WHEN** the window spans the start of British Summer Time
- **THEN** the skipped local hour does not appear and the day has 46 settlement periods

### Requirement: Gaps are shown, not smoothed

Periods with no frame SHALL appear as marked gaps on the timeline, and the map MUST NOT interpolate across them.

#### Scenario: Missing frames

- **WHEN** the history window contains missing periods
- **THEN** the timeline marks them and scrubbing through shows the no-data treatment

### Requirement: Shareable moments

The current time and selection SHALL be encoded in the URL so a link restores the same frame, and a link to a time outside the available window MUST degrade gracefully to the nearest available time with a notice.

#### Scenario: Open a shared link

- **WHEN** someone opens a link to a time inside the window
- **THEN** the same frame and selection appear

#### Scenario: Link outside the window

- **WHEN** the linked time is older than the retained history
- **THEN** the map opens at the earliest available frame with a notice that the requested moment is unavailable

### Requirement: Replay timeline is explicit

In replay mode the timeline SHALL cover the recorded window only, label times as recorded, and offer looping so an unattended demo keeps moving.

#### Scenario: Unattended demo

- **WHEN** replay mode runs with looping enabled and nobody interacts
- **THEN** playback loops the recorded window with the replay marker visible throughout
