# Project Research Summary

**Project:** Gearing Calculator
**Domain:** Interactive Jupyter Notebook — Vehicle Gearing Calculator
**Researched:** 2026-02-02
**Confidence:** MEDIUM-HIGH

## Executive Summary

This project is an interactive Jupyter notebook for visualizing 5-speed transmission shift points, using ipywidgets for inputs and matplotlib for real-time plotting. The core value proposition is the **connected shift line visualization** that shows RPM climbing through each gear and dropping at upshifts — a unique approach compared to traditional gearing calculators that show disconnected gear curves.

The recommended approach follows the mature, "boring" stack: **JupyterLab 4.x + ipywidgets 8.x + matplotlib 3.8+ with ipympl** for the widget backend. This is the established standard for interactive engineering calculators in Jupyter. The architecture follows a clean cell-based structure with separated concerns: calculation functions (pure math), plotting functions (visualization), and widget bindings (interactivity). This pattern enables easy testing and incremental development.

The primary risks are **matplotlib figure accumulation** (memory leaks from creating new figures on each update) and **output widget duplication** (re-running cells creates multiple event handlers). Both are solved by establishing correct patterns in Phase 1: reuse figure objects, use `interactive_output()` for binding, and set `continuous_update=False` on sliders to prevent performance degradation. The domain is well-understood with mature libraries, making this a low-risk project architecturally.

## Key Findings

### Recommended Stack

The stack is mature and straightforward with high confidence. JupyterLab 4.x provides the modern notebook environment, ipywidgets 8.x delivers the interactive controls (sliders, text inputs), and matplotlib 3.8+ with ipympl handles plotting with the widget backend. This combination is specifically designed for slider-driven calculations with immediate visual feedback.

**Core technologies:**
- **JupyterLab 4.x**: Modern notebook environment with full ipywidgets compatibility
- **ipywidgets 8.x**: Standard for Jupyter interactivity, mature API for sliders and numerical inputs
- **matplotlib 3.8+**: Scientific plotting with excellent ipywidgets integration via `%matplotlib widget`
- **ipympl 0.9+**: Required for interactive matplotlib in JupyterLab (provides widget backend)
- **numpy 1.24+**: Efficient array operations for RPM/speed calculations across gear ranges

**Key architectural decision:** Use `interactive_output()` pattern (not `@interact` decorator) for maximum control over layout and clean separation between widget definition and display logic.

**Version warning:** Specific versions based on January 2025 training data; verify with PyPI before installation.

### Expected Features

The feature landscape is well-defined with clear table stakes and differentiators. The core insight: users need to **see** the shift behavior, not just calculate it.

**Must have (table stakes):**
- Input widgets for tire diameter, 5 gear ratios, final drive ratio, engine max RPM
- Single shift RPM slider (simpler than per-gear sliders)
- Speed vs RPM graph with labeled axes (Speed mph, RPM)
- Connected shift line showing RPM climb and drops at upshifts
- Real-time graph updates on any input change
- Pre-loaded defaults (working example on first run)
- Input validation (prevent zero/negative values)

**Should have (competitive differentiators):**
- Annotated shift speeds (text labels at each shift point, e.g., "1-2: 28 mph")
- Visible shift point markers (vertical lines or dots at discontinuities)
- Max speed indicator (top speed at engine max RPM in top gear)
- Gear band highlighting (shaded regions showing speed range per gear)
- RPM band coloring (green/yellow/red based on RPM thresholds)

**Defer (v2+):**
- Per-gear shift RPM control (clutters UI, advanced users can fork notebook)
- Horsepower/torque curves (scope creep into power analysis vs kinematics)
- Downshift calculations (context-dependent, no single correct answer)
- Multiple vehicle comparison (clutters graph)
- Automatic shift point optimization (requires defining optimization criteria)

**Critical anti-feature:** Avoid per-gear shift RPM sliders in v1 — single slider is the differentiator that makes this tool simple and explorable.

### Architecture Approach

The architecture follows a standard 6-cell Jupyter notebook structure with clean separation of concerns: imports, constants, calculation functions (pure math), plotting function, widget definitions, and interactive binding. Data flows immutably from widget values through pure calculation functions to matplotlib rendering, with `interactive_output()` handling the event binding.

**Major components:**
1. **Calculation Layer** (Cell 3): Pure functions (`speed_from_rpm()`, `calculate_shift_points()`) that take parameters and return numpy arrays. No matplotlib, no widgets, fully testable.
2. **Visualization Layer** (Cell 4): `update_plot()` function that receives parameters, calls calculation layer, renders matplotlib figure with formatting and labels.
3. **Presentation Layer** (Cells 5-6): Widget definitions with defaults, `interactive_output()` binding to connect widgets to plotting function, VBox/HBox layout for display.

**Critical pattern:** Reuse figure objects instead of creating new ones on each update. Initialize `fig, ax = plt.subplots()` once, then update data with `line.set_data()` or use `ax.clear()` + redraw. This prevents the most common pitfall (matplotlib figure accumulation causing memory leaks).

**Build order:** Cells 1-2 (imports/constants) → Cell 3 (calculations, testable in isolation) → Cell 4 (plotting, testable with hardcoded values) → Cells 5-6 (widgets/binding for full interactivity).

### Critical Pitfalls

Research identified 6 critical and 4 moderate pitfalls, with clear phase assignments for mitigation.

1. **Matplotlib Figure Accumulation (Memory Leak)** — Creating new `plt.figure()` in update callbacks without closing old figures causes memory exhaustion after 50+ slider moves. **Prevention:** Reuse single figure object, update data with `line.set_data()` or `ax.clear()`. Must be addressed in Phase 1 as core architecture decision.

2. **Output Widget Duplication** — Re-running cells creates multiple widget instances with duplicate event handlers. Slider moves trigger 2x, 3x updates. **Prevention:** Use `clear_output(wait=True)` in Output widget context, or display widgets only once in dedicated cell. Address in Phase 1 to establish correct lifecycle pattern.

3. **Missing Widget Backend** — Default `%matplotlib inline` creates static PNG images; updates stack vertically instead of updating existing plot. **Prevention:** Use `%matplotlib widget` with ipympl, or manually manage display with Output widget. First line of code decision in Phase 1.

4. **continuous_update=True Performance** — Slider fires hundreds of callbacks while dragging, causing massive lag with matplotlib redraws (50-200ms each). **Prevention:** Set `continuous_update=False` on sliders to update only on release. Address in Phase 1; optimize in Phase 2 if smooth drag needed.

5. **State Management Chaos** — Callbacks reference stale global variables or closure values; widgets show one value, plot shows different value. **Prevention:** Use `interactive_output()` pattern to pass all parameters explicitly, or read `widget.value` directly in callbacks. Architecture decision in Phase 1.

6. **Plot Axis Limits Don't Update** — When reusing figure, axis limits from first render persist; new data gets clipped. **Prevention:** Call `ax.relim()` and `ax.autoscale_view()` after updating data. Include in Phase 1 update pattern.

## Implications for Roadmap

Based on research, suggested 3-phase structure aligned with natural dependencies and pitfall mitigation:

### Phase 1: Foundation & Basic Interactivity
**Rationale:** Establish core architecture patterns before adding features. This phase addresses all 6 critical pitfalls by implementing correct figure management, widget lifecycle, and state management from the start. Dependencies are linear (imports → calculations → plotting → widgets), making this the natural starting point.

**Delivers:** Working interactive notebook with basic shift line visualization. User can input vehicle specs, adjust shift RPM with slider, see real-time plot updates.

**Features (from FEATURES.md):**
- All table stakes: input widgets (tire, 5 gears, final drive, max RPM), shift RPM slider, speed vs RPM graph with labeled axes
- Connected shift line showing climbs and drops
- Real-time updates with proper event handling
- Pre-loaded defaults, input validation

**Stack elements (from STACK.md):**
- JupyterLab + ipywidgets + matplotlib + ipympl installation
- `%matplotlib widget` backend setup
- 6-cell notebook structure (imports → constants → calculations → plotting → widgets → binding)

**Architecture (from ARCHITECTURE.md):**
- Pure calculation functions (`speed_from_rpm()`, `calculate_shift_points()`)
- `update_plot()` function with figure reuse pattern
- `interactive_output()` binding with proper parameter passing

**Avoids pitfalls:**
- Pitfall 2 (figure accumulation): Reuse figure object
- Pitfall 1 (widget duplication): Proper display pattern
- Pitfall 3 (missing backend): `%matplotlib widget` setup
- Pitfall 4 (continuous_update): Set `False` on sliders
- Pitfall 6 (state chaos): `interactive_output()` pattern
- Pitfall 10 (axis limits): Include `relim()` + `autoscale_view()`

**Research flag:** Standard patterns, no additional research needed. ipywidgets documentation sufficient.

### Phase 2: Visual Enhancements
**Rationale:** Add differentiating features that make shift behavior obvious and improve usability. Builds on Phase 1 architecture without changing core patterns. These features are independent (annotations, markers, indicators) so can be added incrementally.

**Delivers:** Polished visualization with annotations, markers, and improved layout that makes the tool production-ready.

**Features (from FEATURES.md):**
- Annotated shift speeds (text labels at each shift point)
- Visible shift point markers (vertical lines at discontinuities)
- Max speed indicator (vertical line or text at top gear max)
- Improved layout (VBox/HBox grouping of related inputs)

**Avoids pitfalls:**
- Pitfall 9 (missing initial render): Ensure plot appears on first load
- Pitfall 8 (layout chaos): Group inputs with HBox/VBox, plot beside controls

**Research flag:** Standard matplotlib annotations and layout patterns, no additional research needed.

### Phase 3: Advanced Polish & User Features
**Rationale:** Add features that require more complex rendering or user customization. These are nice-to-have enhancements that differentiate the tool but aren't essential for core functionality. Separating from Phase 2 allows early release after Phase 2.

**Delivers:** Feature-complete tool with advanced visualizations and quality-of-life improvements.

**Features (from FEATURES.md):**
- Gear band highlighting (shaded regions with `axvspan()`)
- RPM band coloring (green/yellow/red line segments)
- Overdrive indicator (mark gears with ratio < 1.0)
- Reset to defaults button
- Export graph as PNG button
- Ratio step display (numerical steps between gears)

**Avoids pitfalls:**
- Pitfall 7 (FloatText validation): Add proper validation for any new text inputs
- UX pitfalls: Add tooltips, units in labels, error handling

**Research flag:** May need matplotlib color segmentation research for RPM band coloring (more complex than standard plotting).

### Phase 4 (Future): Optional Extensions
**Not part of initial roadmap, but documented for future consideration:**
- Preset configurations dropdown (Jeep JK, Tacoma, etc.)
- 6-speed transmission support (variable gear count)
- Crawl ratio calculation, speedometer error calculator
- Export data as CSV

**Explicitly rejected (from FEATURES.md anti-features):**
- Horsepower/torque curves, acceleration calculations, downshift analysis, automatic optimization

### Phase Ordering Rationale

- **Foundation first (Phase 1):** All subsequent phases depend on correct architecture patterns established here. Fixing pitfalls later requires refactoring; fixing them upfront is 10x cheaper.
- **Visual enhancements second (Phase 2):** Annotations and markers are standard matplotlib operations that build on existing plotting function. Independent features allow incremental addition.
- **Advanced polish third (Phase 3):** Color segmentation and custom styling are more complex; separating allows earlier release after Phase 2 if timeline pressures emerge.
- **Dependency flow:** Linear (1 → 2 → 3) with no backtracking. Each phase builds on previous without changing core patterns.

### Research Flags

**Phases with standard patterns (skip research-phase):**
- **Phase 1:** ipywidgets and matplotlib patterns are mature and well-documented. Existing research (STACK.md, ARCHITECTURE.md, PITFALLS.md) covers everything needed.
- **Phase 2:** Standard matplotlib annotations and layout. Official docs sufficient.

**Phases potentially needing focused research:**
- **Phase 3:** RPM band coloring (multi-color line segments) may need matplotlib documentation review or StackOverflow research for cleanest approach. Not critical path, can defer.

**Overall:** This is a low-complexity project with well-established patterns. No deep technical research needed during phase planning — implementation can proceed directly from existing research.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Mature libraries (ipywidgets, matplotlib) with stable APIs; versions based on Jan 2025 training (verify current versions) |
| Features | MEDIUM-HIGH | Table stakes well-defined from domain knowledge; differentiators based on visualization insight; confidence limited by lack of competitor analysis |
| Architecture | HIGH | Standard Jupyter + ipywidgets patterns; cell structure and `interactive_output()` approach well-established; performance estimates should validate during implementation |
| Pitfalls | HIGH | Critical pitfalls (figure accumulation, widget duplication, backend setup) are universal to ipywidgets/matplotlib projects; solutions verified through training knowledge |

**Overall confidence:** MEDIUM-HIGH

The technical approach is sound with high confidence in stack and architecture. Lower confidence in features comes from inability to verify against current (2026) online gearing calculators, but table stakes are clear from domain fundamentals.

### Gaps to Address

**During implementation (not blockers):**
- **Version verification:** Confirm ipywidgets 8.x, matplotlib 3.8+, ipympl 0.9+ are current versions as of February 2026. Check PyPI or official docs.
- **Performance validation:** Confirm 50-100ms matplotlib rendering estimate holds on target hardware. If slower, may need `continuous_update=False` or figure reuse optimization.
- **Feature validation:** After Phase 1 MVP, consider user testing to confirm table stakes categorization. May reveal missing obvious features.

**Not addressed in research (out of scope):**
- **Formula verification:** Research assumes existing spreadsheet formula is correct. Should validate `speed = RPM * π * tire_dia / (ratio * final * 1056)` against automotive engineering sources.
- **Default values:** Pre-loaded defaults (35" tires, 4.56 final drive, etc.) should match realistic vehicle specs.

**Acceptable uncertainty:**
- Exact matplotlib rendering performance (50-100ms is estimate; actual may vary ±50%)
- Whether users prefer `continuous_update=True` vs `False` (can toggle during Phase 2 based on testing)
- Optimal layout (HBox vs VBox arrangements) — iterate during Phase 2

## Sources

### Primary (HIGH confidence)
- **ipywidgets API patterns** (training data through Jan 2025): `interactive_output()`, widget types, layout containers, continuous_update behavior
- **matplotlib integration patterns** (training data): Widget backend (`%matplotlib widget`), figure reuse, axis limit management
- **Jupyter notebook conventions** (training data): Cell structure, import organization, state management

### Secondary (MEDIUM confidence)
- **Automotive gearing calculator domain** (training data): Table stakes features (tire diameter, gear ratios, final drive), common visualization approaches
- **ipywidgets performance characteristics** (training data): continuous_update performance implications, figure accumulation memory leak patterns
- **Engineering calculator UX patterns** (training data): Pre-loaded defaults, real-time updates, input validation expectations

### Limitations
- **No web research:** WebSearch and WebFetch unavailable; could not verify current online gearing calculator features or latest library versions
- **No official docs verification:** Could not cross-reference ipywidgets 8.x or matplotlib 3.8+ documentation for 2025-2026 changes
- **No competitor analysis:** Unable to confirm table stakes categorization against current tools (e.g., grimmjeeper, summitracing calculators)

### Recommended validation
- Check PyPI for current stable versions: `pip index versions ipywidgets matplotlib ipympl`
- Review ipywidgets documentation: https://ipywidgets.readthedocs.io/en/stable/
- Review matplotlib widget backend docs: https://matplotlib.org/stable/users/explain/interactive.html
- Test performance assumptions on target hardware during Phase 1

---
*Research completed: 2026-02-02*
*Ready for roadmap: yes*
