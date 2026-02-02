# Stack Research

**Domain:** Interactive Jupyter Notebook — Engineering Calculator
**Researched:** 2026-02-02
**Overall Confidence:** MEDIUM (based on training data through Jan 2025, unable to verify current versions)

## Executive Summary

For an interactive engineering calculator in Jupyter with widgets and real-time matplotlib visualization, the standard stack is mature and straightforward: **ipywidgets** for interactive controls, **matplotlib** with the **widget backend** for plotting, and **JupyterLab** as the environment. This combination provides the most reliable, widely-supported approach for slider-driven calculations with immediate visual feedback.

**Key recommendation:** Stick with the "boring" standard stack. The ecosystem has converged on ipywidgets + matplotlib for exactly this use case.

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended | Confidence |
|------------|---------|---------|-----------------|------------|
| **JupyterLab** | 4.x | Notebook environment | Modern interface, better extension support than classic Notebook, full ipywidgets compatibility | HIGH |
| **ipywidgets** | 8.x | Interactive UI controls | Standard for Jupyter interactivity, mature API, extensive widget library including sliders, numerical inputs | HIGH |
| **matplotlib** | 3.8+ | Plotting and visualization | Standard scientific plotting, excellent integration with ipywidgets via `%matplotlib widget` backend, no web dependencies | HIGH |
| **numpy** | 1.24+ | Numerical calculations | Required for efficient array operations (RPM/speed calculations across gear ranges) | HIGH |

### Supporting Libraries

| Library | Version | Purpose | When to Use | Confidence |
|---------|---------|---------|-------------|------------|
| **ipympl** | 0.9+ | Matplotlib widget backend | REQUIRED for interactive matplotlib in JupyterLab (provides `%matplotlib widget` magic) | HIGH |
| **pandas** | 2.x | Data structuring (optional) | If you want to display calculation results in tabular form alongside plots | MEDIUM |

### Development Tools

| Tool | Version | Purpose | Why | Confidence |
|------|---------|---------|-----|------------|
| **Python** | 3.9+ | Runtime | ipywidgets 8.x requires Python 3.7+, recommend 3.9+ for stability | HIGH |
| **pip** | Latest | Package manager | Standard Python package installation | HIGH |

## Installation

### Complete Setup

```bash
# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install complete stack
pip install jupyterlab ipywidgets matplotlib ipympl numpy

# Optional: for tabular display
pip install pandas
```

### Minimal Setup (if environment exists)

```bash
pip install ipywidgets matplotlib ipympl numpy
```

### Verify Installation

```python
# Run in notebook cell to verify
import ipywidgets as widgets
import matplotlib.pyplot as plt
import numpy as np
print(f"ipywidgets: {widgets.__version__}")
print(f"matplotlib: {plt.matplotlib.__version__}")
print(f"numpy: {np.__version__}")
```

### Enable Widget Backend

Add this to the first code cell of your notebook:

```python
%matplotlib widget
import ipywidgets as widgets
import matplotlib.pyplot as plt
import numpy as np
```

## Alternatives Considered

| Category | Recommended | Alternative | Why Not | Confidence |
|----------|-------------|-------------|---------|------------|
| **Widgets** | ipywidgets | Jupyter Widgets (old name) | Same library, ipywidgets is current name | HIGH |
| **Widgets** | ipywidgets | Panel (HoloViz) | More complex, designed for dashboards not notebooks, unnecessary for this use case | MEDIUM |
| **Widgets** | ipywidgets | Voila | For converting notebooks to standalone apps, not for in-notebook interactivity | HIGH |
| **Plotting** | matplotlib | Plotly | Heavier (requires JS bundling), overkill for 2D line plots, less seamless with ipywidgets | MEDIUM |
| **Plotting** | matplotlib | Bokeh | Similar to Plotly, adds complexity without benefit for this use case | MEDIUM |
| **Plotting** | matplotlib | Altair | Declarative syntax is verbose for real-time updates, designed for static viz | MEDIUM |
| **Backend** | `%matplotlib widget` (ipympl) | `%matplotlib notebook` | Deprecated in favor of widget backend | HIGH |
| **Backend** | `%matplotlib widget` | `%matplotlib inline` | Static images only, no interactivity | HIGH |
| **Environment** | JupyterLab | Jupyter Notebook Classic | Classic is in maintenance mode, JupyterLab is the future | HIGH |
| **Environment** | JupyterLab | VS Code Jupyter | Good alternative, but JupyterLab is standard for sharing .ipynb files | MEDIUM |
| **Environment** | JupyterLab | Google Colab | Cloud-based, requires internet, less control over environment | MEDIUM |

## What NOT to Use

| Avoid | Why | Use Instead | Confidence |
|-------|-----|-------------|------------|
| `%matplotlib notebook` | Deprecated backend, replaced by widget backend | `%matplotlib widget` with ipympl | HIGH |
| `%matplotlib inline` | Creates static images, no interactivity with widgets | `%matplotlib widget` with ipympl | HIGH |
| `interact_manual` without `interact` | Requires button clicks, not real-time slider updates | `widgets.interact()` or `@widgets.interact` decorator | HIGH |
| Plotly for this use case | Unnecessary complexity, larger dependency footprint, JS bundling issues in some environments | matplotlib with widget backend | MEDIUM |
| Custom JavaScript widgets | Massive development overhead, breaks portability | Built-in ipywidgets (FloatSlider, IntSlider, FloatText, etc.) | HIGH |
| `plt.ion()` with repeated `plt.draw()` | Manual update loop, not idiomatic with ipywidgets | Let ipywidgets handle updates via `widgets.interact()` | HIGH |

## Recommended Architecture

### Widget Pattern: Use `widgets.interactive_output()`

For maximum control and clean separation:

```python
# Define calculation function
def update_plot(shift_rpm, tire_dia, final_drive, gear1, gear2, gear3, gear4, gear5):
    # Clear and redraw plot
    ax.clear()
    # ... calculation and plotting logic
    fig.canvas.draw_idle()

# Create widgets
shift_rpm_slider = widgets.IntSlider(min=2000, max=8000, value=6000, description='Shift RPM')
tire_dia_input = widgets.FloatText(value=26.0, description='Tire Dia (in)')
# ... more widgets

# Link widgets to function
output = widgets.interactive_output(update_plot, {
    'shift_rpm': shift_rpm_slider,
    'tire_dia': tire_dia_input,
    # ... more mappings
})

# Display
display(widgets.VBox([shift_rpm_slider, tire_dia_input, ...]), output)
```

### Alternative Pattern: `@widgets.interact` Decorator

Simpler for straightforward cases:

```python
@widgets.interact(
    shift_rpm=widgets.IntSlider(min=2000, max=8000, value=6000),
    tire_dia=(20.0, 35.0, 0.5),
    # ... more parameters
)
def update_plot(shift_rpm, tire_dia, ...):
    plt.clf()  # Clear figure
    # ... calculation and plotting
```

## Domain-Specific Considerations

### Real-Time Performance

For your use case (5-speed transmission with ~25-50 data points per calculation):
- **Numpy vectorization is sufficient** - No need for optimization libraries
- **matplotlib.pyplot is fast enough** - Single line plot redraws in <50ms
- **Avoid creating new figures** - Reuse existing fig/ax objects

### Widget Types for Engineering Input

| Input Type | Widget | Why |
|------------|--------|-----|
| Shift RPM (variable) | `IntSlider` or `FloatSlider` | User will adjust frequently, slider provides immediate feedback |
| Tire diameter | `FloatText` or `BoundedFloatText` | Precise input, changes infrequently |
| Gear ratios | `FloatText` (5 separate) | Precise values, rarely adjusted during session |
| Final drive | `FloatText` | Precise value, rarely adjusted |
| Engine max RPM | `IntText` | Reference value for validation/display |

### Plot Update Strategy

**Recommended:** Clear and redraw on each update
```python
ax.clear()
ax.plot(speeds, rpms, marker='o')
ax.set_xlabel('Speed (mph)')
ax.set_ylabel('RPM')
fig.canvas.draw_idle()
```

**Not Recommended:** Animated lines with `set_data()`
- Adds complexity
- Matplotlib not optimized for animation in this context
- Clear/redraw is fast enough for this use case

## Version Notes

**IMPORTANT - Confidence Warning:**

The versions listed above are based on training data through January 2025. I was unable to verify current versions via web search or documentation fetch. Before installation:

1. Check PyPI for latest stable versions:
   - `pip index versions ipywidgets`
   - `pip index versions matplotlib`
   - `pip index versions ipympl`

2. Or visit official sources:
   - ipywidgets: https://ipywidgets.readthedocs.io/
   - matplotlib: https://matplotlib.org/
   - JupyterLab: https://jupyterlab.readthedocs.io/

**As of my training (Jan 2025):**
- ipywidgets 8.x is current (8.1.x latest)
- matplotlib 3.8.x is current
- ipympl 0.9.x is current
- JupyterLab 4.x is current

## Known Compatibility Issues

| Issue | Affects | Solution | Confidence |
|-------|---------|----------|------------|
| ipympl not installed | `%matplotlib widget` fails with "No module named 'ipympl'" | Install ipympl explicitly: `pip install ipympl` | HIGH |
| Widget backend in VS Code | May require notebook restart after enabling | Restart Jupyter kernel after first `%matplotlib widget` | MEDIUM |
| Multiple figure updates | Figures stack instead of updating | Reuse fig/ax objects, use `ax.clear()` before redrawing | HIGH |
| Slider lag with complex plots | Updates feel sluggish | Use `continuous_update=False` on sliders to update only on release | HIGH |

## Quick Start Template

```python
# Cell 1: Setup
%matplotlib widget
import ipywidgets as widgets
import matplotlib.pyplot as plt
import numpy as np

# Cell 2: Create figure (once)
fig, ax = plt.subplots(figsize=(10, 6))

# Cell 3: Define update function
def update_plot(shift_rpm, tire_dia, final_drive, g1, g2, g3, g4, g5):
    gears = [g1, g2, g3, g4, g5]

    # Calculate speed at each shift point for each gear
    speeds = []
    rpms = []

    # ... your calculation logic ...

    ax.clear()
    ax.plot(speeds, rpms, 'b-o', linewidth=2, markersize=8)
    ax.set_xlabel('Speed (mph)', fontsize=12)
    ax.set_ylabel('RPM', fontsize=12)
    ax.set_title('Shift Point Analysis', fontsize=14)
    ax.grid(True, alpha=0.3)
    fig.canvas.draw_idle()

# Cell 4: Create widgets
shift_slider = widgets.IntSlider(
    value=6000, min=2000, max=8000, step=100,
    description='Shift RPM:', continuous_update=True
)
tire_input = widgets.FloatText(value=26.0, description='Tire Dia:')
# ... more widgets ...

# Cell 5: Link and display
ui = widgets.VBox([shift_slider, tire_input, ...])
out = widgets.interactive_output(update_plot, {
    'shift_rpm': shift_slider,
    'tire_dia': tire_input,
    # ...
})
display(ui, out)
```

## Sources

**Confidence Note:** All information based on training data through January 2025. Unable to verify with current official documentation due to tool access limitations.

**Recommended verification sources:**
- ipywidgets documentation: https://ipywidgets.readthedocs.io/en/stable/
- matplotlib documentation: https://matplotlib.org/stable/
- ipympl repository: https://github.com/matplotlib/ipympl
- JupyterLab documentation: https://jupyterlab.readthedocs.io/

**Training data sources (implicit):**
- Standard scientific Python stack conventions
- ipywidgets API patterns (stable since v7.x)
- matplotlib integration best practices
- Jupyter ecosystem evolution

## Confidence Assessment

| Component | Confidence | Rationale |
|-----------|------------|-----------|
| ipywidgets as standard | HIGH | Mature library, no serious alternatives for Jupyter-native widgets |
| matplotlib for 2D plots | HIGH | Standard scientific plotting library, excellent Jupyter integration |
| ipympl requirement | HIGH | Well-documented requirement for interactive matplotlib in JupyterLab |
| Specific versions | MEDIUM | Based on Jan 2025 training, unable to verify current releases |
| Architecture patterns | HIGH | Established best practices, stable API patterns |
| Performance assumptions | HIGH | Use case (simple line plots) well within matplotlib capabilities |

## Next Steps for Validation

1. After installation, verify versions match or exceed recommendations
2. Test widget backend functionality with simple slider + plot
3. If using VS Code instead of JupyterLab, verify ipympl compatibility
4. Check matplotlib backend with: `matplotlib.get_backend()` (should show 'module://ipympl.backend_nbagg')
