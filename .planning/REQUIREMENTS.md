# Requirements: Gearing Calculator

**Defined:** 2026-02-02
**Core Value:** The shift point visualization — one connected line climbing through each gear and dropping at each upshift — must update instantly as the shift RPM slider moves.

## v1 Requirements

### Inputs

- [x] **INPUT-01**: User can set tire diameter in inches via widget (default: 35)
- [x] **INPUT-02**: User can set 5 gear ratios via widgets (defaults: 3.72, 2.20, 1.50, 1.00, 0.79)
- [x] **INPUT-03**: User can set final drive ratio via widget (default: 4.56)
- [x] **INPUT-04**: User can set engine max RPM via widget (default: 5500)
- [x] **INPUT-05**: User can adjust shift RPM via slider (single value for all gears)
- [x] **INPUT-06**: All widgets pre-loaded with spreadsheet defaults on first run

### Graph

- [x] **GRAPH-01**: Graph displays speed (mph) on X-axis and RPM on Y-axis
- [x] **GRAPH-02**: Connected shift line climbs through each gear and drops vertically at each upshift
- [x] **GRAPH-03**: Graph updates in real time when any widget value changes
- [x] **GRAPH-04**: Annotated shift speeds show text labels at each shift point (e.g., "1-2: 28 mph")
- [x] **GRAPH-05**: Shift point markers visually indicate each gear transition
- [x] **GRAPH-06**: Max speed indicator shows top speed at max RPM in top gear
- [x] **GRAPH-07**: Gear band highlighting shows shaded speed regions per gear
- [x] **GRAPH-08**: RPM band coloring shows green/yellow/red line segments based on RPM

### Info Display

- [x] **INFO-01**: Ratio step display shows ratio between consecutive gears (e.g., 1st/2nd = 1.69)

## v2 Requirements

### Quality of Life

- **QOL-01**: Input validation prevents zero/negative values
- **QOL-02**: Reset to defaults button restores all widgets to original values
- **QOL-03**: Export graph as PNG button
- **QOL-04**: Overdrive indicator marks gears with ratio < 1.0

## Out of Scope

| Feature | Reason |
|---------|--------|
| Per-gear shift RPM sliders | Clutters UI; single slider keeps tool simple and explorable |
| Horsepower/torque curves | Scope creep into power analysis; this is purely kinematic |
| Downshift calculations | Context-dependent; no single correct answer |
| Multiple vehicle comparison | Clutters the graph; single vehicle at a time |
| Automatic shift point optimization | Requires defining optimization criteria; user explores manually |
| 6-speed or variable gear count | Fixed at 5-speed for v1 simplicity |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| INPUT-01 | Phase 1 | Complete |
| INPUT-02 | Phase 1 | Complete |
| INPUT-03 | Phase 1 | Complete |
| INPUT-04 | Phase 1 | Complete |
| INPUT-05 | Phase 1 | Complete |
| INPUT-06 | Phase 1 | Complete |
| GRAPH-01 | Phase 1 | Complete |
| GRAPH-02 | Phase 1 | Complete |
| GRAPH-03 | Phase 1 | Complete |
| GRAPH-04 | Phase 2 | Complete |
| GRAPH-05 | Phase 2 | Complete |
| GRAPH-06 | Phase 2 | Complete |
| GRAPH-07 | Phase 3 | Complete |
| GRAPH-08 | Phase 3 | Complete |
| INFO-01 | Phase 3 | Complete |

**Coverage:**
- v1 requirements: 15 total
- Mapped to phases: 15
- Unmapped: 0

---
*Requirements defined: 2026-02-02*
*Last updated: 2026-02-02 after all phases complete*
