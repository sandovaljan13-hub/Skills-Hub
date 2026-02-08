# Charts & Visual Generation

This reference covers chart selection, generation, styling, and accuracy verification. You generate all charts — the user should never have to create charts themselves.

Use pptxgenjs native charts (BAR, LINE, PIE, DOUGHNUT, SCATTER, BUBBLE, RADAR) as documented in `/mnt/skills/public/pptx/pptxgenjs.md` for standard chart types. For chart types that pptxgenjs doesn't support natively (waterfall, Sankey, combination charts), generate them as high-resolution PNG images using matplotlib/seaborn, then embed in the slide.

---

## Chart Selection Guide

The right chart type depends on what relationship you're trying to show, not what the data looks like.

| Relationship to show | Chart type | Example use case |
|---------------------|-----------|-----------------|
| Comparing categories | Column/Bar (pptxgenjs BAR) | Revenue by region, department spend comparison |
| Trends over time | Line (pptxgenjs LINE) | Monthly revenue, quarterly growth trajectory |
| Part of whole (2-3 segments) | Donut (pptxgenjs DOUGHNUT) | Market share, revenue mix. Use donut over pie — cleaner center space for a callout number. |
| Part of whole (4+ segments) | Stacked bar (pptxgenjs BAR) | Budget allocation, cost breakdown by category |
| Variance / bridge analysis | Waterfall (matplotlib → PNG) | EBITDA bridge, budget variance, revenue walk. This is the chart C-levels ask for most. |
| Correlation | Scatter (pptxgenjs SCATTER) | Spend vs. outcome, price vs. volume |
| Single KPI spotlight | Large number callout (text, not a chart) | "42% YoY growth" at 60-72pt with a subtitle |

When in doubt between chart types, ask yourself: "What comparison am I trying to make obvious?" If it's "which is bigger," use bars. If it's "how did it change," use a line. If it's "what's the breakdown," use a donut or stacked bar. If it's "what drove the change from A to B," use a waterfall.

---

## Chart Styling

Default chart styling in pptxgenjs looks dated. These adjustments bring charts to executive-presentation quality:

```javascript
slide.addChart(pres.charts.BAR, chartData, {
  x: 0.5, y: 1, w: 9, h: 4, barDir: "col",

  // Match your deck's color palette — never use defaults
  chartColors: ["1E2761", "C9A84C", "7A8DA6"],

  // Clean background
  chartArea: { fill: { color: "FFFFFF" }, roundedCorners: true },

  // Muted axis labels — they shouldn't compete with the data
  catAxisLabelColor: "64748B",
  valAxisLabelColor: "64748B",

  // Subtle value-axis grid only
  valGridLine: { color: "E2E8F0", size: 0.5 },
  catGridLine: { style: "none" },

  // Data labels on bars (only when ≤8 data points)
  showValue: true,
  dataLabelPosition: "outEnd",
  dataLabelColor: "1E293B",

  // Hide legend for single series
  showLegend: false,
});
```

**Key principles:**
- Color palette must match the deck theme. The chart should look like it belongs in the presentation, not like it was pasted from Excel.
- Remove horizontal grid lines from bar charts. Keep subtle grid lines on line charts only if they help readability.
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
