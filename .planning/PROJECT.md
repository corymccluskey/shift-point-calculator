# Gearing Calculator

## What This Is

An interactive Jupyter notebook that calculates and visualizes shift points for a vehicle with a 5-speed manual transmission. Users input tire size, gear ratios, final drive ratio, and engine max RPM via widgets, then use a slider to adjust shift RPM and see a real-time graph showing the RPM-vs-speed curve with vertical drops at each shift point.

## Core Value

The shift point visualization — one connected line climbing through each gear and dropping at each upshift — must update instantly as the shift RPM slider moves.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Widget inputs for tire diameter (inches), 5 transmission ratios, final drive ratio, and engine max RPM
- [ ] Shift RPM slider that controls where upshifts happen (same RPM for all gears)
- [ ] Graph with speed (mph) on X-axis and RPM on Y-axis
- [ ] Connected shift line: climbs through each gear, drops vertically at each shift point to the next gear's RPM
- [ ] Pre-loaded defaults from existing spreadsheet (35" tires, 3.72/2.20/1.50/1.00/0.79 ratios, 4.56 final drive, 5500 max RPM)
- [ ] Graph updates in real time when any input changes

### Out of Scope

- Horsepower/torque curves — this is purely kinematic (speed/RPM relationship)
- Multiple vehicles or comparison mode — single vehicle at a time
- Data export or saving — notebook is the artifact
- Downshift calculations — upshifts only

## Context

- Existing spreadsheet (`gearing calc.xlsx`) has the core math: `speed = RPM * pi * tire_dia / (gear_ratio * final_drive * 1056)`
- Spreadsheet column C computes ratio steps between gears (determines RPM drop magnitude on upshift)
- This is for the CarTruckMoto project — personal vehicle gearing analysis
- Jupyter notebook chosen for interactive widget support (ipywidgets)

## Constraints

- **Format**: Jupyter notebook (.ipynb) — must run standalone with standard scientific Python stack
- **Interactivity**: ipywidgets for all inputs and sliders — no code editing required to use
- **Gears**: Fixed at 5-speed transmission

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single shift RPM for all gears | Simpler interaction, covers primary use case | — Pending |
| Connected shift line (not separate gear curves) | Clearer visualization of actual RPM behavior during acceleration | — Pending |
| All inputs as widgets | No code editing needed to explore different setups | — Pending |

---
*Last updated: 2026-02-02 after initialization*
