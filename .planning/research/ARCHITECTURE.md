# Architecture Research

**Domain:** Interactive Jupyter Notebook — Engineering Calculator
**Researched:** 2026-02-02
**Confidence:** MEDIUM

> Note: Based on established patterns for ipywidgets and matplotlib integration. WebSearch unavailable; relying on training knowledge of stable, mature technologies (ipywidgets 7.x/8.x, matplotlib 3.x patterns).

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│ JUPYTER NOTEBOOK CELL STRUCTURE                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  [Cell 1: Imports]                                           │
│    └─> numpy, matplotlib, ipywidgets                         │
│                                                              │
│  [Cell 2: Configuration & Constants]                         │
│    └─> Default values, conversion factors                    │
│                                                              │
│  [Cell 3: Core Calculation Functions]                        │
│    └─> Pure functions (input → output)                       │
│    └─> speed_from_rpm(), calculate_shift_points()            │
│                                                              │
│  [Cell 4: Plotting Function]                                 │
│    └─> update_plot(params) - creates/updates figure          │
│                                                              │
│  [Cell 5: Widget Definitions]                                │
│    └─> FloatSlider, FloatText, IntSlider widgets             │
│                                                              │
│  [Cell 6: Interactive Display]                               │
│    └─> ipywidgets.interactive_output() OR                    │
│    └─> @ipywidgets.interact decorator                        │
│    └─> Binds widgets → plotting function                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘

DATA FLOW:

  User Interaction
       ↓
  Widget Value Change
       ↓
  Observer/Interactive Callback Triggered
       ↓
  Extract Current Widget Values
       ↓
  Call Calculation Functions
       ↓
  Generate Data Arrays (speeds[], rpms[])
       ↓
  Update Matplotlib Figure
       ↓
  Display Refreshes Automatically
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|---------------|------------------------|
| **Imports Cell** | Load all dependencies | `import numpy as np`<br>`import matplotlib.pyplot as plt`<br>`from ipywidgets import interact, FloatSlider, VBox, HBox` |
| **Constants/Config** | Define defaults, magic numbers | Default tire size, gear ratios, conversion factor (1056) |
| **Calculation Functions** | Pure math operations | `def speed_from_rpm(rpm, tire_dia, gear_ratio, final_drive)`<br>Returns speed in mph |
| **Data Preparation** | Build arrays for plotting | Generate speed/RPM arrays for each gear, handle shift drops |
| **Plotting Function** | Render matplotlib figure | `def update_plot(**params)` - clear, plot, format axes, return figure |
| **Widget Definitions** | Create input controls | FloatSlider for shift RPM, FloatText for ratios, etc. |
| **Interactive Binding** | Connect widgets to plot | `ipywidgets.interactive_output(update_plot, widget_dict)` |
| **Layout/Display** | Organize UI | VBox/HBox to arrange widgets + output |

## Recommended Notebook Structure

### Cell-by-Cell Layout

```python
# ═══════════════════════════════════════════════════════════
# CELL 1: Imports
# ═══════════════════════════════════════════════════════════
import numpy as np
import matplotlib.pyplot as plt
from ipywidgets import (
    interact, interactive, interactive_output,
    FloatSlider, FloatText, IntSlider,
    VBox, HBox, Label, Output
)
from IPython.display import display

# ═══════════════════════════════════════════════════════════
# CELL 2: Constants and Default Values
# ═══════════════════════════════════════════════════════════
# Pre-loaded defaults from existing spreadsheet
DEFAULT_TIRE_DIA = 35.0  # inches
DEFAULT_GEAR_RATIOS = [3.72, 2.20, 1.50, 1.00, 0.79]  # gears 1-5
DEFAULT_FINAL_DRIVE = 4.56
DEFAULT_MAX_RPM = 5500
DEFAULT_SHIFT_RPM = 4500

# Conversion constant from formula
CONVERSION_FACTOR = 1056  # From speed = RPM * π * tire_dia / (ratio * final * 1056)

# ═══════════════════════════════════════════════════════════
# CELL 3: Core Calculation Functions
# ═══════════════════════════════════════════════════════════
def speed_from_rpm(rpm, tire_dia, gear_ratio, final_drive):
    """Calculate vehicle speed in mph from engine RPM."""
    return rpm * np.pi * tire_dia / (gear_ratio * final_drive * CONVERSION_FACTOR)

def calculate_shift_points(tire_dia, gear_ratios, final_drive, max_rpm, shift_rpm):
    """
    Generate arrays for plotting shift line.
    Returns: (speeds, rpms) as numpy arrays
    """
    speeds = []
    rpms = []

    for i, gear_ratio in enumerate(gear_ratios):
        # Start RPM for this gear (idle for first gear, drop from previous shift)
        if i == 0:
            start_rpm = 1000  # idle
        else:
            # Calculate RPM after upshift from previous gear
            prev_speed = speed_from_rpm(shift_rpm, tire_dia, gear_ratios[i-1], final_drive)
            start_rpm = prev_speed * gear_ratio * final_drive * CONVERSION_FACTOR / (np.pi * tire_dia)

        # End RPM is shift point (or max for final gear)
        end_rpm = shift_rpm if i < len(gear_ratios) - 1 else max_rpm

        # Generate points for this gear's climb
        rpm_range = np.linspace(start_rpm, end_rpm, 50)
        speed_range = speed_from_rpm(rpm_range, tire_dia, gear_ratio, final_drive)

        speeds.extend(speed_range)
        rpms.extend(rpm_range)

    return np.array(speeds), np.array(rpms)

# ═══════════════════════════════════════════════════════════
# CELL 4: Plotting Function
# ═══════════════════════════════════════════════════════════
def update_plot(tire_dia, gear1, gear2, gear3, gear4, gear5,
                final_drive, max_rpm, shift_rpm):
    """
    Generate and return matplotlib figure with shift line.
    This function is called every time any widget changes.
    """
    # Clear previous plot
    plt.clf()

    # Prepare data
    gear_ratios = [gear1, gear2, gear3, gear4, gear5]
    speeds, rpms = calculate_shift_points(
        tire_dia, gear_ratios, final_drive, max_rpm, shift_rpm
    )

    # Create figure
    fig, ax = plt.subplots(figsize=(12, 6))

    # Plot connected shift line
    ax.plot(speeds, rpms, 'b-', linewidth=2, label='Shift Line')

    # Format
    ax.set_xlabel('Speed (mph)', fontsize=12)
    ax.set_ylabel('RPM', fontsize=12)
    ax.set_title('Transmission Shift Points', fontsize=14, fontweight='bold')
    ax.grid(True, alpha=0.3)
    ax.set_ylim(0, max_rpm * 1.1)

    # Add shift RPM line
    ax.axhline(y=shift_rpm, color='r', linestyle='--', alpha=0.5, label=f'Shift at {shift_rpm} RPM')
    ax.legend()

    plt.tight_layout()
    return fig

# ═══════════════════════════════════════════════════════════
# CELL 5: Widget Definitions
# ═══════════════════════════════════════════════════════════
# Input widgets with defaults
widget_tire_dia = FloatText(
    value=DEFAULT_TIRE_DIA,
    description='Tire Diameter (in):',
    style={'description_width': '150px'}
)

widget_gear1 = FloatText(value=DEFAULT_GEAR_RATIOS[0], description='1st Gear:', style={'description_width': '150px'})
widget_gear2 = FloatText(value=DEFAULT_GEAR_RATIOS[1], description='2nd Gear:', style={'description_width': '150px'})
widget_gear3 = FloatText(value=DEFAULT_GEAR_RATIOS[2], description='3rd Gear:', style={'description_width': '150px'})
widget_gear4 = FloatText(value=DEFAULT_GEAR_RATIOS[3], description='4th Gear:', style={'description_width': '150px'})
widget_gear5 = FloatText(value=DEFAULT_GEAR_RATIOS[4], description='5th Gear:', style={'description_width': '150px'})

widget_final_drive = FloatText(
    value=DEFAULT_FINAL_DRIVE,
    description='Final Drive:',
    style={'description_width': '150px'}
)

widget_max_rpm = IntSlider(
    value=DEFAULT_MAX_RPM,
    min=3000, max=8000, step=100,
    description='Max RPM:',
    style={'description_width': '150px'}
)

widget_shift_rpm = IntSlider(
    value=DEFAULT_SHIFT_RPM,
    min=2000, max=7000, step=100,
    description='Shift RPM:',
    style={'description_width': '150px'},
    continuous_update=True  # CRITICAL: enables real-time updates as slider moves
)

# ═══════════════════════════════════════════════════════════
# CELL 6: Interactive Display Assembly
# ═══════════════════════════════════════════════════════════
# Create output widget for plot
output = interactive_output(
    update_plot,
    {
        'tire_dia': widget_tire_dia,
        'gear1': widget_gear1,
        'gear2': widget_gear2,
        'gear3': widget_gear3,
        'gear4': widget_gear4,
        'gear5': widget_gear5,
        'final_drive': widget_final_drive,
        'max_rpm': widget_max_rpm,
        'shift_rpm': widget_shift_rpm
    }
)

# Layout widgets
input_layout = VBox([
    Label(value='Vehicle Parameters', style={'font_weight': 'bold'}),
    widget_tire_dia,
    Label(value='Transmission Ratios', style={'font_weight': 'bold'}),
    widget_gear1, widget_gear2, widget_gear3, widget_gear4, widget_gear5,
    Label(value='Performance Parameters', style={'font_weight': 'bold'}),
    widget_final_drive,
    widget_max_rpm,
    widget_shift_rpm
])

# Display UI
display(HBox([input_layout, output]))
```

### Build Order Recommendation

1. **Cells 1-2 First** (Imports + Constants)
   - No dependencies, quick validation that environment works

2. **Cell 3 Second** (Calculation Functions)
   - Can test in isolation: `speed_from_rpm(3000, 35, 3.72, 4.56)` should return ~15.8 mph
   - Validates core math before visualization complexity

3. **Cell 4 Third** (Plotting Function)
   - Test with hardcoded values: `update_plot(35, 3.72, 2.20, 1.50, 1.00, 0.79, 4.56, 5500, 4500)`
   - Validates visualization before widget binding

4. **Cells 5-6 Last** (Widgets + Binding)
   - Only after plotting function works correctly
   - Enables interactive testing of full system

## Architectural Patterns

### Pattern 1: interactive_output() vs @interact Decorator

**Recommended: `interactive_output()` (used in structure above)**

**Why:**
- Separates widget definition from display logic
- Allows custom layout (VBox/HBox arrangement)
- More control over output placement
- Easier to test plotting function independently

```python
# GOOD: Separation of concerns
output = interactive_output(update_plot, widget_dict)
display(HBox([controls, output]))
```

**Alternative: `@interact` decorator**

```python
# Works but less flexible
@interact(shift_rpm=(2000, 7000, 100))
def plot_shifts(shift_rpm):
    # plotting code here
    pass
```

**Trade-off:** `@interact` is faster to prototype but harder to customize layout.

### Pattern 2: Widget Update Strategy

**Critical Setting: `continuous_update=True`**

```python
widget_shift_rpm = IntSlider(
    value=4500,
    continuous_update=True  # Updates while dragging, not just on release
)
```

**When to use:**
- `continuous_update=True`: Real-time sliders (shift RPM in this case)
- `continuous_update=False`: Heavy computations or slow renders

**For this project:** Use `True` for shift_rpm slider (core interaction), can use `False` for other widgets if performance becomes an issue.

### Pattern 3: Figure Management

**Option A: Create new figure each time (RECOMMENDED for simplicity)**

```python
def update_plot(params):
    plt.clf()  # Clear previous
    fig, ax = plt.subplots(figsize=(12, 6))
    # ... plotting code
    return fig
```

**Option B: Reuse figure and update data (performance optimization)**

```python
# Global figure initialization
fig, ax = plt.subplots(figsize=(12, 6))
line, = ax.plot([], [])

def update_plot(params):
    speeds, rpms = calculate_shift_points(params)
    line.set_data(speeds, rpms)
    ax.relim()
    ax.autoscale_view()
    fig.canvas.draw()
    return fig
```

**Recommendation:** Start with Option A (simpler, no state management). Only move to Option B if performance is slow (>200ms update time).

### Pattern 4: Data Flow Architecture

**CRITICAL: Immutable data flow**

```
Widget Values (mutable state)
       ↓
Extract to parameters (immutable)
       ↓
Pure calculation functions (no side effects)
       ↓
Return new arrays
       ↓
Render to new/updated figure
```

**Anti-pattern to avoid:**

```python
# BAD: Global mutable state
current_speeds = []
current_rpms = []

def update_plot(params):
    global current_speeds, current_rpms
    current_speeds = calculate_speeds(params)  # Mutating global
    # ...
```

**Why bad:** Hard to debug, race conditions, stale data issues.

### Pattern 5: Matplotlib Backend Consideration

**Default backend (inline):**

```python
%matplotlib inline
```

**For better interactivity (if needed):**

```python
%matplotlib widget  # Enables zoom, pan within notebook
```

**Recommendation:** Start with `inline` (simplest). Only switch to `widget` if users need zoom/pan capabilities (unlikely for this calculator).

## Data Flow

### Widget Change Event Flow

```
1. User drags shift_rpm slider
       ↓
2. ipywidgets detects value change
       ↓
3. interactive_output observer fires callback
       ↓
4. Callback extracts ALL current widget values
       ↓
5. Calls update_plot(tire_dia=35, gear1=3.72, ..., shift_rpm=4200)
       ↓
6. update_plot() calls calculate_shift_points()
       ↓
7. calculate_shift_points() loops through gears:
       - For each gear: calculate start_rpm (from previous shift drop)
       - Generate 50 points from start_rpm to shift_rpm
       - Convert each RPM to speed using speed_from_rpm()
       ↓
8. Returns (speeds[], rpms[]) as numpy arrays
       ↓
9. update_plot() renders matplotlib figure:
       - ax.plot(speeds, rpms, 'b-')
       - Add formatting, labels, grid
       ↓
10. Figure displays in Output widget
       ↓
11. User sees updated graph (~50-100ms total)
```

### Critical Performance Consideration

**Bottleneck:** Matplotlib rendering, not calculation.

- Calculation (numpy arrays): ~1ms
- Matplotlib render: ~50-100ms

**Optimization strategy (if needed):**
- Reduce figure size: `figsize=(10, 5)` instead of `(12, 6)`
- Reduce data points: `np.linspace(start, end, 30)` instead of 50
- Use `continuous_update=False` for non-critical widgets

**For this project:** 100ms update is acceptable for interactive feel. No optimization needed unless testing reveals lag.

## Anti-Patterns

### Anti-Pattern 1: Mixing Computation and Display

**BAD:**
```python
def update_plot(params):
    # Computation mixed with plotting
    speeds = []
    for i in range(5):
        rpm = 3000
        while rpm < shift_rpm:
            speed = rpm * 3.14 * tire_dia / ...  # Formula inline
            speeds.append(speed)
            rpm += 100
    plt.plot(speeds)
```

**GOOD:**
```python
def calculate_shift_points(params):
    # Pure calculation
    return speeds, rpms

def update_plot(params):
    speeds, rpms = calculate_shift_points(params)
    plt.plot(speeds, rpms)  # Pure display
```

**Why:** Separation enables testing calculations without rendering.

### Anti-Pattern 2: Not Using Vectorization

**BAD:**
```python
speeds = []
for rpm in range(1000, 5000, 100):
    speed = speed_from_rpm(rpm, tire_dia, gear_ratio, final_drive)
    speeds.append(speed)
```

**GOOD:**
```python
rpms = np.linspace(1000, 5000, 50)
speeds = speed_from_rpm(rpms, tire_dia, gear_ratio, final_drive)  # Vectorized
```

**Why:** 10-100x faster, cleaner code, enables numpy broadcasting.

### Anti-Pattern 3: Over-Complicating Widget Binding

**BAD:**
```python
def on_tire_change(change):
    update_plot()

def on_gear1_change(change):
    update_plot()

# ... 9 separate observers

widget_tire_dia.observe(on_tire_change, 'value')
widget_gear1.observe(on_gear1_change, 'value')
# ...
```

**GOOD:**
```python
output = interactive_output(update_plot, widget_dict)
# Automatically observes all widgets
```

**Why:** Less code, less error-prone, standard pattern.

### Anti-Pattern 4: Hard-Coding Magic Numbers

**BAD:**
```python
speed = rpm * 3.14159 * tire_dia / (gear_ratio * final_drive * 1056)
```

**GOOD:**
```python
# At top of notebook
CONVERSION_FACTOR = 1056
PI = np.pi

speed = rpm * PI * tire_dia / (gear_ratio * final_drive * CONVERSION_FACTOR)
```

**Why:** Easier to verify against formula, clear intent, maintainable.

### Anti-Pattern 5: Forgetting to Handle Edge Cases

**BAD:**
```python
def calculate_shift_points(...):
    for i, gear_ratio in enumerate(gear_ratios):
        start_rpm = # ... assumes previous gear exists
```

**GOOD:**
```python
def calculate_shift_points(...):
    for i, gear_ratio in enumerate(gear_ratios):
        if i == 0:
            start_rpm = 1000  # First gear starts at idle
        else:
            start_rpm = # ... calculate from previous shift
```

**Why:** Handles first gear cleanly, no index errors.

### Anti-Pattern 6: Not Setting Axis Limits

**BAD:**
```python
plt.plot(speeds, rpms)
# Axes rescale every update, causing jarring jumps
```

**GOOD:**
```python
plt.plot(speeds, rpms)
ax.set_ylim(0, max_rpm * 1.1)  # Fixed Y range
ax.set_xlim(0, max_speed * 1.05)  # Fixed X range
```

**Why:** Stable view reduces cognitive load during real-time updates.

## Component Boundaries

### Clear Separation of Responsibilities

```
┌──────────────────────────────────────────────────────────┐
│ PRESENTATION LAYER                                       │
│ - Widget definitions (Cell 5)                            │
│ - Layout/display (Cell 6)                                │
│ - No business logic                                      │
└──────────────────────────────────────────────────────────┘
                          ↕
┌──────────────────────────────────────────────────────────┐
│ VISUALIZATION LAYER                                      │
│ - update_plot() function (Cell 4)                        │
│ - Matplotlib rendering                                   │
│ - Formatting, labels, styling                            │
│ - Calls calculation layer, no direct widget access       │
└──────────────────────────────────────────────────────────┘
                          ↕
┌──────────────────────────────────────────────────────────┐
│ CALCULATION LAYER                                        │
│ - Pure functions (Cell 3)                                │
│ - speed_from_rpm(), calculate_shift_points()             │
│ - Numpy operations                                       │
│ - No matplotlib, no widgets                              │
└──────────────────────────────────────────────────────────┘
                          ↕
┌──────────────────────────────────────────────────────────┐
│ DATA LAYER                                               │
│ - Constants (Cell 2)                                     │
│ - Default values                                         │
│ - Conversion factors                                     │
└──────────────────────────────────────────────────────────┘
```

**Testing implications:**

- **Calculation layer:** Testable with `assert speed_from_rpm(3000, 35, 3.72, 4.56) == pytest.approx(15.8)`
- **Visualization layer:** Testable with `fig = update_plot(params); assert len(fig.axes[0].lines) == 1`
- **Presentation layer:** Manual testing only (interactive)

## Build Order and Dependencies

### Phase 1: Foundation (Cells 1-2)
**No dependencies**
- Import all libraries
- Define constants
- Validate environment

**Validation:**
```python
# Run in new cell
print(f"NumPy version: {np.__version__}")
print(f"Matplotlib version: {plt.matplotlib.__version__}")
print(f"Default tire: {DEFAULT_TIRE_DIA}")
```

### Phase 2: Core Logic (Cell 3)
**Depends on:** Phase 1 only

**Validation:**
```python
# Test calculation
test_speed = speed_from_rpm(3000, 35, 3.72, 4.56)
print(f"Speed at 3000 RPM in 1st gear: {test_speed:.2f} mph")
# Expected: ~15.8 mph

# Test shift points
speeds, rpms = calculate_shift_points(35, [3.72, 2.20, 1.50, 1.00, 0.79], 4.56, 5500, 4500)
print(f"Generated {len(speeds)} points")
print(f"Max speed: {speeds[-1]:.2f} mph")
```

### Phase 3: Visualization (Cell 4)
**Depends on:** Phases 1-2

**Validation:**
```python
# Test plotting with hardcoded values
fig = update_plot(35, 3.72, 2.20, 1.50, 1.00, 0.79, 4.56, 5500, 4500)
plt.show()
# Expect: Connected line with 5 segments and 4 drops
```

### Phase 4: Interactivity (Cells 5-6)
**Depends on:** Phases 1-3

**Validation:**
- Run cells 5-6
- Drag shift_rpm slider
- Confirm graph updates in real-time
- Change tire diameter, confirm recalculation

### Dependency Graph

```
Cell 1 (Imports) ────┬──> Cell 3 (Calculations)
                     │
Cell 2 (Constants) ──┴──> Cell 4 (Plotting) ──> Cell 5 (Widgets) ──> Cell 6 (Display)
```

**Critical path:** 1 → 2 → 3 → 4 → 5 → 6 (linear, no parallelization possible)

## Scalability Considerations

### Current Scope (5 gears, single vehicle)
- **Data size:** ~250 points (5 gears × 50 points/gear)
- **Update frequency:** Every widget change (~1-10 Hz)
- **Rendering time:** ~50-100ms

**No performance concerns expected.**

### If Scope Expands

| Concern | Threshold | Solution |
|---------|-----------|----------|
| More gears (e.g., 6-speed, 10-speed) | >10 gears | Reduce points/gear from 50 to 30 |
| Comparison mode (multiple vehicles) | >3 vehicles | Use `blit=True` for matplotlib animation |
| Add torque curves | >1000 data points | Switch to `%matplotlib widget` + optimize rendering |
| Export/save functionality | N/A | Add `plt.savefig()` button widget |

**For current project:** No scalability architecture needed.

## Technology Constraints

### Jupyter Notebook Limitations

1. **State management:** Cells can run out of order
   - **Mitigation:** Clear "Run All" instruction in notebook header
   - **Mitigation:** No global mutable state

2. **Widget state persistence:** Widgets reset on kernel restart
   - **Mitigation:** Defaults loaded from constants (Cell 2)
   - **Mitigation:** Not a concern for exploratory tool

3. **Browser memory:** Large plots can cause slowdown
   - **Mitigation:** Limited to ~250 points, no concern

4. **Matplotlib backend:** `inline` backend creates new image each update
   - **Mitigation:** 50-100ms is acceptable latency
   - **Alternative:** `widget` backend if needed

### Library Version Compatibility

**Assumed stable versions:**
- `ipywidgets >= 7.0` (continuous_update, interactive_output)
- `matplotlib >= 3.0` (modern API)
- `numpy >= 1.15` (standard operations)

**Verification needed:** Check actual versions in deployment environment.

## Sources

**Confidence note:** This architecture is based on established patterns for Jupyter notebooks with ipywidgets and matplotlib. These are mature, stable technologies with well-documented best practices.

**Training knowledge sources (as of Jan 2025):**
- ipywidgets official documentation patterns (interactive_output, widget types)
- Matplotlib API and performance characteristics
- Jupyter notebook cell organization conventions
- NumPy vectorization patterns

**Limitations:**
- Could not verify current (2026) ipywidgets version features via WebSearch
- Could not cross-reference with recent blog posts or tutorials
- Assumed stable API (likely valid for mature libraries)

**Verification recommended:**
- Check ipywidgets documentation for any new features in 2025-2026
- Confirm matplotlib `widget` backend behavior if needed
- Validate continuous_update performance on target hardware

**Confidence assessment:**
- **Core patterns (cells, interactive_output, calculation separation):** HIGH - fundamental architecture unchanged since ipywidgets 7.x
- **Performance estimates (50-100ms):** MEDIUM - based on typical hardware, should be validated
- **Specific API details:** MEDIUM - stable APIs but version-specific features not verified
- **Build order recommendations:** HIGH - based on dependency analysis
