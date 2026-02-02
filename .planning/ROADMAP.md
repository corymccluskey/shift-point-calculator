# Roadmap: Gearing Calculator

## Overview

This roadmap delivers an interactive Jupyter notebook for visualizing 5-speed transmission shift points. Phase 1 establishes the foundation with all input widgets and the core connected shift line visualization that updates in real time. Phase 2 adds visual enhancements (annotations, markers, max speed indicator) that make shift behavior obvious. Phase 3 completes the tool with advanced polish features (gear bands, RPM coloring, ratio step display).

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Foundation & Basic Interactivity** - Core calculation engine, widget inputs, and connected shift line visualization
- [x] **Phase 2: Visual Enhancements** - Annotations, markers, and max speed indicator
- [x] **Phase 3: Advanced Polish** - Gear bands, RPM coloring, and ratio step display

## Phase Details

### Phase 1: Foundation & Basic Interactivity
**Goal**: User can input vehicle specs and see a real-time connected shift line visualization
**Depends on**: Nothing (first phase)
**Requirements**: INPUT-01, INPUT-02, INPUT-03, INPUT-04, INPUT-05, INPUT-06, GRAPH-01, GRAPH-02, GRAPH-03
**Success Criteria** (what must be TRUE):
  1. User can set tire diameter, 5 gear ratios, final drive ratio, and max RPM via widgets with pre-loaded defaults
  2. User can adjust shift RPM with slider and see the shift line update in real time
  3. Graph displays speed (mph) vs RPM with connected line that climbs through gears and drops vertically at shift points
  4. Graph updates instantly when any input changes (no lag, no figure accumulation)
**Plans**: TBD

Plans:
- [ ] 01-01: TBD during phase planning

### Phase 2: Visual Enhancements
**Goal**: Shift behavior is obvious through annotations, markers, and max speed indicator
**Depends on**: Phase 1
**Requirements**: GRAPH-04, GRAPH-05, GRAPH-06
**Success Criteria** (what must be TRUE):
  1. Each shift point shows text label with transition and speed (e.g., "1-2: 28 mph")
  2. Shift points have visible markers (vertical lines or dots) at each gear transition
  3. Max speed at engine redline in top gear is clearly indicated (vertical line or text annotation)
**Plans**: TBD

Plans:
- [ ] 02-01: TBD during phase planning

### Phase 3: Advanced Polish
**Goal**: Production-ready tool with gear band highlighting, RPM coloring, and ratio step display
**Depends on**: Phase 2
**Requirements**: GRAPH-07, GRAPH-08, INFO-01
**Success Criteria** (what must be TRUE):
  1. Graph shows shaded regions (gear bands) indicating speed range for each gear
  2. Shift line segments are color-coded (green/yellow/red) based on RPM thresholds
  3. Ratio step display shows numerical ratio between consecutive gears (e.g., 1st/2nd = 1.69)
**Plans**: TBD

Plans:
- [ ] 03-01: TBD during phase planning

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & Basic Interactivity | 1/1 | Complete | 2026-02-02 |
| 2. Visual Enhancements | 1/1 | Complete | 2026-02-02 |
| 3. Advanced Polish | 1/1 | Complete | 2026-02-02 |
