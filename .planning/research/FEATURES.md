# Feature Research

**Domain:** Vehicle Gearing Calculator / Shift Point Visualizer
**Researched:** 2026-02-02
**Confidence:** MEDIUM

**Confidence note:** WebSearch and WebFetch unavailable. Research based on training knowledge of automotive gearing calculators and Jupyter notebook UX patterns. Core engineering features have HIGH confidence (well-established domain). Differentiator features have MEDIUM confidence (based on comparative analysis patterns). Interactive notebook UX has HIGH confidence (ipywidgets patterns).

---

## Feature Landscape

### Table Stakes (Users Expect These)

Features that must exist or the tool feels incomplete/broken. Missing any of these makes users abandon the tool.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| **Input: Tire diameter** | Foundation of speed calculation; every gearing calc has this | Low | Standard widget (float input with validation) |
| **Input: Gear ratios (all gears)** | Core identity of gearing calculator | Low | 5 separate inputs or array widget |
| **Input: Final drive ratio** | Essential multiplier for transmission output | Low | Single float widget |
| **Input: Engine max RPM** | Defines upper boundary of usable RPM range | Low | Single float/int widget |
| **Visualization: Speed vs RPM** | Must SEE the relationship, not just calculate | Medium | Line plot, speed on X-axis |
| **Real-time graph updates** | Interactive tools update instantly on input change | Medium | ipywidgets `observe()` pattern |
| **Pre-loaded defaults** | Users need working example to understand tool | Low | Hardcoded initial values |
| **Clear axis labels** | Speed (mph) and RPM must be labeled with units | Low | Matplotlib axis formatting |
| **Visible shift points** | Must show WHERE shifts occur on graph | Medium | Vertical lines or markers at discontinuities |
| **Input validation** | Prevent crashes from impossible values (zero ratios, negative tires) | Medium | Widget constraints + error handling |

### Differentiators (Competitive Advantage)

Features that set this tool apart from basic gearing calculators. Not expected, but significantly increase value.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **Single shift RPM slider** | Explore shift timing impact instantly without per-gear complexity | Low | Current design — simpler than per-gear sliders |
| **Connected shift line** | Shows actual RPM behavior (climb + drop) vs separate gear curves | Medium | Unique visualization — most tools show disconnected curves |
| **RPM drop visualization** | Vertical drops make RPM loss at shifts obvious | Medium | Emergent from connected line — highlights inefficiency |
| **Annotated shift speeds** | Label each shift with exact speed (e.g., "1-2: 28 mph") | Medium | Text annotations on graph at shift points |
| **Gear band highlighting** | Shaded regions showing speed range of each gear | Medium | Matplotlib `axvspan()` with transparency |
| **Max speed indicator** | Show vehicle top speed at engine max RPM in top gear | Low | Vertical line or text label at rightmost point |
| **RPM band coloring** | Visual feedback: green (safe), yellow (approaching redline), red (over max) | Medium | Color segments of line based on RPM thresholds |
| **Ratio step display** | Show numerical ratio step between gears (e.g., "2nd/1st = 0.59") | Low | Calculated value displayed as text widget or table |
| **Speed @ specific RPM markers** | Show speed at common RPMs (3000, 4000, 5000) across all gears | Medium | Grid lines or markers at fixed RPM values |
| **Instant reset button** | Return to defaults without reloading notebook | Low | Button widget that resets all inputs |
| **Export graph as image** | Save current visualization for documentation/sharing | Low | Matplotlib savefig triggered by button |
| **Preset configurations** | Dropdown of common vehicles/setups (e.g., "Jeep JK", "Toyota Tacoma") | Medium | Requires curating preset data |
| **Overdrive indicator** | Highlight when gear ratio < 1.0 (overdrive) | Low | Visual marker or text label on overdrive gears |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem useful but add complexity without proportional value. Explicitly scope these out.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|--------------|-----------------|-------------|
| **Per-gear shift RPM sliders** | "Want different shift points for each gear" | 5 sliders clutters UI; most use cases want uniform shift strategy | Single slider with note that advanced users can fork notebook |
| **Horsepower/torque curves** | "Want to optimize shift points for power" | Requires engine dyno data; scope creep into power analysis vs kinematic analysis | Out of scope — state this is kinematic only |
| **Downshift calculations** | "What about downshifting?" | Downshift strategy is context-dependent (braking, cornering); no single "right" answer | Focus on upshift (acceleration) use case |
| **Multiple vehicle comparison** | "Compare my truck to my car" | Clutters graph; confuses which input affects which line | One vehicle at a time; users can run notebook twice |
| **Automatic shift point optimization** | "Find optimal shift RPM for me" | Requires optimization criteria (fuel economy? acceleration?); no universally correct answer | Manual slider lets user explore; document common strategies |
| **Imperial/metric unit toggle** | "I need km/h" | Adds conditional logic throughout; most US automotive enthusiasts use mph | Hardcode mph; document conversion formula in notebook text |
| **Database of vehicles** | "Pre-load my vehicle's specs" | Maintenance burden; data accuracy liability; scope creep | Provide examples in markdown; users input their own specs |
| **Acceleration time calculations** | "How fast 0-60?" | Requires power/weight ratio, drag, traction; well beyond kinematics | Out of scope — different tool domain |
| **3D visualization** | "Show RPM, speed, and time" | Overkill for 2D relationship; harder to read | 2D graph is clearer and sufficient |
| **Animation of shifting** | "Animate the RPM climbing and dropping" | Cool but gimmicky; slider already provides interactivity | Static graph with instant updates is clearer |

---

## Feature Dependencies

### Core Dependency Tree

```
Inputs (tire, ratios, final drive, max RPM)
  └─> Speed calculation
       └─> Shift point calculation (requires shift RPM slider)
            └─> Visualization (requires matplotlib figure)
                 ├─> Connected shift line (core visualization)
                 ├─> Shift point markers (depends on shift calculation)
                 ├─> Annotated shift speeds (depends on shift points)
                 ├─> Gear band highlighting (depends on shift points)
                 ├─> RPM band coloring (depends on line segments)
                 └─> Max speed indicator (depends on top gear + max RPM)
```

### Implementation Order Implications

1. **Must have first:** Input widgets, core math, basic line plot
2. **Add next:** Shift RPM slider, connected line with vertical drops
3. **Polish phase:** Annotations, highlighting, color coding

**Critical path:** Input widgets → calculation → shift line → real-time updates. Everything else is enhancement.

---

## MVP Definition

### Launch With (v1.0)

**Goal:** Usable tool that demonstrates core value proposition.

#### Must Have
- [ ] Input widgets: tire diameter, 5 gear ratios, final drive, max RPM
- [ ] Shift RPM slider (single value, all gears)
- [ ] Speed vs RPM graph with labeled axes (Speed mph, RPM)
- [ ] Connected shift line (climbs in each gear, drops at shifts)
- [ ] Pre-loaded defaults (35" tires, 3.72/2.20/1.50/1.00/0.79, 4.56 final drive, 5500 max RPM)
- [ ] Real-time graph updates on any input change
- [ ] Input validation (prevent zero/negative values)

#### Nice to Have
- [ ] Annotated shift speeds (text labels at each shift point)
- [ ] Max speed indicator (vertical line or text at top gear max)
- [ ] Visible shift point markers (vertical lines or dots at discontinuities)

**Success criteria:** User can input their vehicle specs, adjust shift RPM, and immediately see the RPM-vs-speed relationship with shift behavior.

---

### Add After Validation (v1.x)

Features to add after core tool is used and validated.

#### v1.1: Visual Polish
- [ ] Gear band highlighting (shaded regions for each gear)
- [ ] RPM band coloring (green/yellow/red based on RPM thresholds)
- [ ] Overdrive indicator (mark gears with ratio < 1.0)
- [ ] Improved legend (show which gear is which on graph)

#### v1.2: Enhanced Usability
- [ ] Reset to defaults button
- [ ] Ratio step display (show numerical steps between gears)
- [ ] Speed @ specific RPM markers (grid lines at 3000, 4000, 5000 RPM)
- [ ] Export graph button (save PNG)

#### v1.3: Power User Features
- [ ] Preset configurations dropdown (Jeep JK, Tacoma, etc.)
- [ ] Grid lines toggle
- [ ] Zoom/pan controls (matplotlib toolbar)
- [ ] Custom RPM thresholds for color bands

**Decision points:** Add these only if v1.0 sees regular use. Don't over-engineer before validation.

---

### Future Consideration (v2.0+)

Features to consider if tool proves valuable and warrants expansion.

#### Potential Additions
- [ ] 4-speed or 6-speed transmission support (variable gear count)
- [ ] Crawl ratio calculation (1st gear × final drive)
- [ ] Speedometer error calculator (tire size changes)
- [ ] Export data as CSV (speed/RPM table)
- [ ] Comparison mode (overlay two configurations)
- [ ] Per-gear shift RPM control (advanced mode toggle)

#### Explicit Rejections
- ❌ Horsepower/torque curves (different tool domain)
- ❌ Acceleration calculations (requires additional data)
- ❌ Downshift analysis (context-dependent, no clear use case)
- ❌ Automatic optimization (requires defining optimization criteria)
- ❌ Multi-vehicle database (maintenance burden)

**Philosophy:** Keep tool focused on kinematic visualization. Resist scope creep into power analysis or optimization.

---

## Domain-Specific Feature Insights

### Automotive Gearing Domain

**Table stakes for gearing calculators:**
- Tire size input (diameter or circumference)
- All gear ratios (not just a subset)
- Final drive ratio (often forgotten by beginners)
- Speed-to-RPM or RPM-to-speed calculation

**Common mistakes in gearing tools:**
- Showing only individual gear curves (disconnected) → misses shift behavior
- Requiring engine power data when doing pure kinematics → scope creep
- Assuming all shifts happen at redline → unrealistic for street driving
- Not validating inputs → crashes on zero ratios

**What differentiates good tools:**
- Shift behavior visualization (the RPM drop is critical insight)
- Interactive exploration (not just static calculation)
- Pre-loaded examples (lower barrier to understanding)

### Interactive Notebook UX Domain

**Table stakes for ipywidgets tools:**
- All parameters as widgets (no code editing required)
- Real-time updates (not "run this cell again")
- Sensible defaults (working example on first run)
- Clear visual feedback (graph changes are obvious)

**Common mistakes in notebook UX:**
- Too many widgets → overwhelming (keep under 10 for MVP)
- Sliders without value display → user doesn't know exact number
- No input validation → crashes feel like user's fault
- Graph doesn't auto-scale → clipped data

**What differentiates good notebook tools:**
- Instant visual feedback (no lag, no "run cell" buttons)
- Progressive disclosure (simple by default, advanced optional)
- Self-documenting (markdown explains what widgets do)
- Export capability (users want to save results)

---

## Feature Complexity Assessment

### Low Complexity (< 1 hour implementation)
- Input widgets (standard ipywidgets)
- Pre-loaded defaults (hardcoded values)
- Basic line plot (matplotlib)
- Axis labels and title
- Input validation (min/max constraints)
- Reset button
- Max speed indicator

### Medium Complexity (1-4 hours implementation)
- Connected shift line (calculating discontinuities)
- Real-time updates (widget observation logic)
- Annotated shift speeds (text positioning)
- Gear band highlighting (coordinate calculation)
- RPM band coloring (segmented line colors)
- Visible shift point markers
- Export graph button

### High Complexity (> 4 hours implementation)
- Preset configurations (data structure + UI)
- Comparison mode (dual plotting logic)
- Variable gear count (dynamic widget generation)
- Advanced zoom/pan (matplotlib interaction)
- Per-gear shift RPM (UI complexity + calculation logic)

**MVP should focus on Low + essential Medium features.**

---

## Sources

**Confidence levels:**
- Automotive gearing calculator features: MEDIUM (based on training knowledge of domain patterns; not verified with current tools)
- Jupyter notebook UX patterns: HIGH (ipywidgets patterns are well-established)
- Feature categorization: MEDIUM (based on domain analysis; not verified with user research)

**Source limitations:**
- WebSearch unavailable → could not verify current online gearing calculator features
- WebFetch unavailable → could not access ipywidgets documentation for latest patterns
- No competitor analysis performed → feature landscape based on domain knowledge

**Recommendations for validation:**
- Test with actual users to confirm table stakes categorization
- Review existing online gearing calculators (e.g., grimmjeeper, summitracing tools)
- Check ipywidgets documentation for current best practices (https://ipywidgets.readthedocs.io/)

**Known gaps:**
- Unable to verify what modern (2026) gearing calculators include
- Unable to verify current ipywidgets best practices
- Feature complexity estimates are based on general Jupyter/matplotlib experience

**Next steps for research:**
- User testing with v1.0 MVP to validate table stakes assumptions
- Competitor feature analysis (when web access available)
- ipywidgets documentation review for advanced patterns
