# Charts & Visual Generation

This reference covers chart selection, generation, styling, and accuracy verification. You generate all charts — the user should never have to create charts themselves.

Use pptxgenjs native charts (BAR, LINE, PIE, DOUGHNUT, SCATTER, BUBBLE, RADAR) as documented in `/mnt/skills/public/pptx/pptxgenjs.md` for standard chart types. For chart types that pptxgenjs doesn't support natively (waterfall, Sankey, combination charts), generate them as high-resolution PNG images using matplotlib/seaborn, then embed in the slide.

---

## Visual Decision Framework: Argument-to-Chart Mapping

In a decision deck, you don't choose a chart based on the data shape — you choose it based on the **argument the slide is making**. Every evidence slide has a Financial Action Title that states a financial implication. The chart's only job is to make that argument visually undeniable.

Before selecting a chart type, identify the argument type from the slide's Action Title, then follow the mandate below. These are not suggestions — they are strict mappings.

| Argument type | Strategic question it answers | Mandated chart | Notes |
|---------------|------------------------------|----------------|-------|
| **Variance / Explaining a gap** | "Why did we miss/beat the target?" | **Waterfall** (matplotlib → PNG) | EBITDA bridges, budget variance, revenue walks. Shows the sequential drivers from starting point to ending point. |
| **Contrast / Ranking options** | "Which is best?" or "Where should we invest?" | **Sorted Horizontal Bar** (pptxgenjs BAR, `barDir: "bar"`) | Always sorted by value, largest at top. Never unsorted. Horizontal so labels are readable without rotation. |
| **Trend / Trajectory** | "Is it getting better or worse?" | **Line with reference line** (pptxgenjs LINE) | Must include a trendline, target line, or benchmark line. A line without context is just data — the reference line provides the "so what." |
| **Composition / Breakdown** | "Where is the money going?" | **Donut** (2-3 segments) or **Stacked Bar** (4 segments) | **Strictly max 4 categories.** Group everything else into "Other." More than 4 categories creates visual noise that undermines the argument. Use donut for 2-3 segments (cleaner center for callout number), stacked bar for exactly 4. |
| **Single KPI spotlight** | "What's the headline number?" | **Large number callout** (text, not a chart) | 60-72pt number with a subtitle. No chart needed — the number is the visual. |
| **Correlation** | "Are these two things related?" | **Scatter** (pptxgenjs SCATTER) | Use sparingly. Only when proving a causal or correlated relationship is central to the slide's argument. |

When in doubt, ask yourself: "What is the argument in my Action Title?" If the title says "X erodes $Y" → that's a variance, use a waterfall. If the title says "Region A outperforms" → that's a contrast, use a sorted horizontal bar. If the title says "trend is accelerating" → use a line with trendline. The argument dictates the chart, never the reverse.

---

## Signal-to-Noise: Maximize Data Ink

Every pixel of ink on a chart must earn its place by communicating data. If a visual element doesn't help the audience understand the argument faster, strip it.

**Mandatory removals:**
- **Horizontal gridlines on bar charts** — remove entirely. Data labels on the bars carry the values; gridlines are redundant noise.
- **Legends on single-series charts** — remove. If there's only one data series, the legend restates the title.
- **Value axis labels** when data labels are shown on bars/points — the axis becomes redundant. Remove to reduce clutter.
- **3D effects, shadows, gradients** — never, on any chart element. These are decoration, not data.
- **Chart borders and outlines** — remove. Charts float cleanly on the slide background.
- **Top, right, and left axis lines (spines)** — remove. Keep only the baseline (x-axis) for grounding. The waterfall code already does this with `ax.spines[['top', 'right']].set_visible(False)` — apply the same principle to all charts.

**Retained for readability:**
- **Subtle gridlines on line charts only** — light grid (`color: "E2E8F0", size: 0.5`) aids trend reading where the audience needs to trace values across time.
- **Legends on multi-series charts** (2+ series) — position at bottom or right, never floating over data.
- **Axis labels** when no data labels are present (e.g., scatter plots).

---

## The Highlight Rule

Every chart must direct the audience's eye to the single data point that supports the slide's Financial Action Title. This is non-negotiable — a chart where all bars are the same color forces the audience to read and interpret. A chart with one highlighted bar tells them instantly where to look.

**The rule:**
1. Identify the specific data point, bar, or line segment referenced in the Financial Action Title
2. Apply the deck's **accent color** to that element only
3. Set all other data points to a **muted neutral** (`#CBD5E1` slate-gray or equivalent from the deck's muted palette)
4. The highlighted element is the argument. Everything else is context.

**pptxgenjs implementation — per-bar highlighting:**

```javascript
// Action Title: "APAC's 34% Growth Delivers $2.1M Incremental EBITDA — Double Down"
// APAC is the third bar — highlight it, mute the rest
slide.addChart(pres.charts.BAR, chartData, {
  x: 0.5, y: 1.2, w: 9, h: 4, barDir: "bar",

  // Highlight Rule: only the argument bar gets the accent color
  chartColors: ["CBD5E1", "CBD5E1", "1E2761", "CBD5E1", "CBD5E1"],
  //             North     East      APAC ←    South     West

  chartArea: { fill: { color: "FFFFFF" }, roundedCorners: true },
  catAxisLabelColor: "64748B",
  valAxisLabelColor: "64748B",
  valGridLine: { style: "none" },   // Signal-to-Noise: no gridlines on bar charts
  catGridLine: { style: "none" },
  showValue: true,
  dataLabelPosition: "outEnd",
  dataLabelColor: "1E293B",
  showLegend: false,
});
```

**matplotlib implementation — waterfall highlight on key driver:**

```python
# Highlight the bar that represents the key driver in the Action Title
# Example: "Pricing Uplift Drives $1.4M of $2.1M EBITDA Beat"
bar_colors = []
highlight_index = 2  # The "Pricing Uplift" bar
for i, v in enumerate(values):
    if i == 0 or i == len(values) - 1:
        bar_colors.append(colors['total'])        # Totals keep their own color
    elif i == highlight_index:
        bar_colors.append(colors['highlight'])     # Key driver gets accent color
    elif v >= 0:
        bar_colors.append('#CBD5E1')               # Other increases muted
    else:
        bar_colors.append('#F1A9A0')               # Other decreases muted (light red)
```

Add `'highlight'` to the waterfall `colors` dict (e.g., `colors = {'total': '#1E2761', 'increase': '#CBD5E1', 'decrease': '#F1A9A0', 'highlight': '#C9A84C', 'connector': '#94A3B8'}`).

---

## Chart Styling

Default chart styling in pptxgenjs looks dated. These adjustments bring charts to decision-deck quality, aligned with the Signal-to-Noise and Highlight Rule standards above.

**Bar/Column charts (the most common in decision decks):**

```javascript
slide.addChart(pres.charts.BAR, chartData, {
  x: 0.5, y: 1.2, w: 9, h: 4, barDir: "bar",

  // Highlight Rule: accent color on the argument bar, muted on the rest
  chartColors: ["CBD5E1", "CBD5E1", "1E2761", "CBD5E1"],

  // Clean background
  chartArea: { fill: { color: "FFFFFF" }, roundedCorners: true },

  // Muted axis labels — they shouldn't compete with the data
  catAxisLabelColor: "64748B",
  valAxisLabelColor: "64748B",

  // Signal-to-Noise: no gridlines on bar charts
  valGridLine: { style: "none" },
  catGridLine: { style: "none" },

  // Data labels on bars (only when ≤8 data points)
  showValue: true,
  dataLabelPosition: "outEnd",
  dataLabelColor: "1E293B",

  // Hide legend for single series
  showLegend: false,
});
```

**Line charts (for trend/trajectory arguments):**

```javascript
slide.addChart(pres.charts.LINE, chartData, {
  x: 0.5, y: 1.2, w: 9, h: 4,

  chartColors: ["1E2761"],  // Primary line in accent color
  lineSize: 2.5,

  chartArea: { fill: { color: "FFFFFF" }, roundedCorners: true },
  catAxisLabelColor: "64748B",
  valAxisLabelColor: "64748B",

  // Line charts retain subtle gridlines for trend reading
  valGridLine: { color: "E2E8F0", size: 0.5 },
  catGridLine: { style: "none" },

  showLegend: false,
});
// Add a target/benchmark line as a second series in a muted color ("CBD5E1")
// or use slide.addShape for a horizontal reference line
```

**Key principles:**
- Color palette must match the deck theme. The chart should look like it belongs in the presentation, not like it was pasted from Excel.
- Highlight Rule always applies — one accent element, everything else muted.
- Include legend only when there are 2+ data series. Position at bottom or right.
- Minimum 10pt for axis labels, 12pt for data labels.
- Charts should be wider than tall (roughly 16:9 or 3:2 proportions). Taller-than-wide charts feel cramped on a widescreen slide.

---

## Waterfall Charts

EBITDA bridges, budget variances, and revenue walks are the chart type C-level audiences request most, and pptxgenjs doesn't support them natively. Generate as matplotlib PNG and embed.

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import numpy as np

def create_waterfall(categories, values, title, colors, filepath, figsize=(10, 5.5)):
    """
    categories: list of labels (first = starting total, last = ending total)
    values: list of numeric values (positive = increase, negative = decrease)
    colors: dict with 'increase', 'decrease', 'total', 'connector' hex colors (include #)
    """
    cumulative = np.zeros(len(values))
    cumulative[0] = values[0]
    for i in range(1, len(values)):
        if i == len(values) - 1:
            cumulative[i] = values[i]
        else:
            cumulative[i] = cumulative[i-1] + values[i]

    bottoms = np.zeros(len(values))
    for i in range(1, len(values)):
        if i == len(values) - 1:
            bottoms[i] = 0
        elif values[i] >= 0:
            bottoms[i] = cumulative[i-1]
        else:
            bottoms[i] = cumulative[i]

    bar_colors = []
    for i, v in enumerate(values):
        if i == 0 or i == len(values) - 1:
            bar_colors.append(colors['total'])
        elif v >= 0:
            bar_colors.append(colors['increase'])
        else:
            bar_colors.append(colors['decrease'])

    fig, ax = plt.subplots(figsize=figsize)
    ax.bar(categories, [abs(v) for v in values], bottom=bottoms, color=bar_colors, width=0.6)

    for i in range(len(values) - 2):
        ax.plot([i - 0.3, i + 1.3], [cumulative[i], cumulative[i]],
                color=colors['connector'], linewidth=0.8, linestyle='--')

    for i, (v, b) in enumerate(zip(values, bottoms)):
        label_y = b + abs(v) / 2
        ax.text(i, label_y, f"${abs(v):,.0f}", ha='center', va='center',
                fontsize=10, fontweight='bold', color='white')

    ax.set_title(title, fontsize=14, fontweight='bold', pad=15)
    ax.spines[['top', 'right']].set_visible(False)
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, p: f"${x:,.0f}"))
    plt.tight_layout()
    plt.savefig(filepath, dpi=200, bbox_inches='tight', transparent=False)
    plt.close()
```

Save waterfall PNGs to `/home/claude/charts/` and embed with `slide.addImage({ path: ... })`.

---

## Accuracy Verification

After generating every chart, print the exact values being plotted and cross-check against the auditor's Verified Metrics:

```python
print(f"Chart: {chart_title}")
print(f"Labels: {labels}")
print(f"Values: {values}")
print(f"Sum check: {sum(values)}")
# Cross-reference against audit output
```

If any chart value doesn't match the verified data, fix it before embedding. This applies equally to anchoring calculations — every comparison, equivalence, or projection that appears on a slide must have a computed verification logged alongside the chart code.

The reason this matters beyond accuracy: if the user ever needs to update the deck with new data, having a clean verification log means they (or you) can trace exactly where every number came from and what assumptions were used.
