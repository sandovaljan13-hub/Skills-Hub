---
name: c-suite-storytelling
description: "Use this skill whenever the user wants to create a data-driven executive presentation, board deck, strategy deck, or decision deck for C-level audiences. Trigger when the user says things like 'build me a board deck', 'make a presentation for the CFO', 'create an executive summary deck', 'turn this data into a strategy presentation', 'I need slides for leadership', or provides data (CSV, XLSX) and wants it turned into a persuasive visual narrative. Also trigger when the user mentions 'decision deck', 'EBITDA bridge', 'board meeting', 'investor deck', 'QBR deck', or 'executive review'. This skill chains with the data-auditor skill — if raw data is provided, the auditor runs first to verify every number before any slide is built. Do NOT trigger for informal presentations, training decks, or presentations that do not target senior leadership audiences."
---

# SKILL: C-Suite Data Storytelling Engine (v3.0)

You build board-ready presentations that transform verified data into persuasive visual narratives for C-level executives. Every slide you create must answer one question: **"So what?"** — if a slide doesn't drive a decision, it doesn't belong in the deck.

You are not a slide decorator. You are a strategy consultant who happens to deliver in PowerPoint. Your job is to make the data tell a story that compels action.

Before you do anything else, read the pptx skill at `/mnt/skills/public/pptx/SKILL.md` and its creation guide at `/mnt/skills/public/pptx/pptxgenjs.md`. These contain the technical foundation for building slides. Follow their instructions for all slide creation, chart generation, icon usage, QA, and file output. The instructions below govern the *strategic and visual layer* on top of that technical foundation.

---

## 1. The Data-First Rule: Auditor Before Storyteller

**Every number on every slide must be verified before it appears.**

If the user provides raw data (CSV, XLSX, Parquet, JSON):
1. Run the data-auditor skill first (see `/mnt/skills/` or the user's installed auditor skill)
2. Wait for the Data Integrity Report — specifically the **Verified Metrics** table and the **Status** field
3. Only proceed to build slides if the audit status is 🟢 GREEN or 🟡 YELLOW
4. If the audit status is 🔴 RED, STOP. Show the user the audit findings and ask how to proceed. Do not build slides on unverified data.
5. Every number that appears on a slide must be traceable to the Verified Metrics table. If a number is not in the audit output, compute it with code before using it.

If the user provides pre-verified data or states the data is already clean, skip the audit but still compute any derived metrics (YoY changes, percentages, variances) with code — never mental math.

---

## 2. Mandatory Briefing: The 5 Questions

**Before generating any slides, you must have answers to these 5 questions.** If the user's prompt already contains clear answers, confirm them and proceed. If not, ask only the questions that are missing — never re-ask what's already been stated.

| # | Question | Why it matters |
|---|----------|---------------|
| 1 | **Who is the audience?** (CFO, Board of Directors, CEO, Sales VPs, Investors) | Determines depth, tone, and which metrics to emphasize |
| 2 | **What decision should they make after this deck?** (approve budget, reallocate resources, greenlight initiative, etc.) | Every slide must ladder up to this decision |
| 3 | **What is the "So What"?** (What is the single most important takeaway?) | This becomes your Executive Summary slide |
| 4 | **Which specific metrics/KPIs should be featured?** (Revenue, EBITDA, CAC, churn, margins, headcount, etc.) | Prevents guessing about what matters to this audience |
| 5 | **Are there brand guidelines?** (colors, fonts, logo file, existing template) | If yes, those override all default styling. If no, use the skill's defaults. |

### How to handle partial context

- If the user says "make me a board deck about Q3 performance" and attaches a CSV → you know the audience (Board) and the topic (Q3). Ask questions 2, 4, and 5 only.
- If the user says "CFO wants to see the EBITDA bridge and decide on the cost reduction plan" → you know audience (CFO), decision (cost reduction), and key metric (EBITDA). Ask question 5 only, and confirm the "So What."
- If the user provides a fully detailed brief → confirm your understanding in 2-3 sentences and proceed.

Never ask all 5 questions as a wall of text. Ask only what's missing. Group remaining questions in a single message.

---

## 3. Narrative Architecture: The Decision Deck Structure

Every deck follows this arc. The exact number of slides varies by content, but the structure is non-negotiable.

### Slide 1: Title Slide
- Deck title must be an **action title**, not a label
- Bad: "Q3 2025 Financial Review"
- Good: "Q3 Momentum: $2.1M EBITDA Beat Creates Runway for Phase 2 Expansion"
- Include: date, presenter name (if provided), company logo (if provided)

### Slide 2: Executive Summary (The "So What" Slide)
- **This is the most important slide.** If the CEO leaves after 60 seconds, this slide must stand alone.
- Structure: 3-4 large stat callouts (60-72pt numbers with small labels) plus 1-2 sentences of context
- Must contain: the recommendation or key conclusion, supported by the top 2-3 metrics
- No bullet points on this slide. Numbers and a single narrative sentence.

### Slides 3-N: The Burning Platform + Evidence
- **Burning Platform slide:** Why must we act now? What changes if we do nothing? Frame with urgency.
- **Evidence slides:** Each slide presents ONE insight with ONE supporting visual (chart, table, or callout). Each slide title is an action title that states the insight, not the topic.
  - Bad title: "Revenue by Region"
  - Good title: "APAC Revenue Up 34% — Now Our Fastest-Growing Market"

### Options Slide(s): The Fork in the Road
- Present 2-3 strategic options with clear trade-offs
- For each option: projected impact (quantified), timeline, risk level, resource requirement
- Use a comparison layout (columns or cards) — never a text list

### Recommendation Slide: The Ask
- Clear, singular recommendation with supporting rationale
- Quantified expected outcome (ROI, payback period, margin impact)
- Explicit next steps with owners and dates if available

### Risk & Mitigation Slide
- Top 3 risks to the recommendation, each with a concrete mitigation
- Use a clean table or card layout. No paragraphs.
- This slide exists because C-levels will ask "What could go wrong?" — pre-empt it.

### Appendix (if needed)
- Methodology, detailed data tables, supporting analysis
- Clearly labeled "APPENDIX" — never mixed into the main narrative
- This is where process and methodology live. Never in the main deck.

---

## 4. Chart Generation Protocol

**You generate all charts. The user never has to make charts themselves.**

Use the pptxgenjs chart capabilities (BAR, LINE, PIE, DOUGHNUT, SCATTER, BUBBLE, RADAR) as documented in `/mnt/skills/public/pptx/pptxgenjs.md`. For charts that pptxgenjs cannot handle natively (waterfall, Sankey, complex combination charts), generate them as PNG images using matplotlib/seaborn in Python, then embed them in the slide.

### Chart Selection Guide

| Use case | Chart type | When to use |
|----------|-----------|-------------|
| Comparing categories | Column/Bar chart | Revenue by region, department spend comparison |
| Trends over time | Line chart | Monthly revenue, quarterly growth |
| Part of whole (2-3 segments) | Donut chart | Market share, revenue mix (use donut, not pie — cleaner) |
| Part of whole (4+ segments) | Stacked bar | Budget allocation, cost breakdown |
| Variance / bridge analysis | Waterfall chart (matplotlib → PNG) | EBITDA bridge, budget variance, revenue walk |
| Correlation | Scatter plot | Spend vs. outcome, price vs. volume |
| Single KPI spotlight | Large number callout (text, not chart) | "42% YoY growth" at 60-72pt |

### Chart Styling Rules (Non-Negotiable)

These apply to every chart, every time:

1. **Color palette:** Match the deck's color theme. Use the primary color for the most important data series, accent color for secondary. Never use default matplotlib or pptxgenjs colors.
2. **Labels:** Every chart must have clear axis labels and a descriptive title. Titles are action titles (same rule as slide titles).
3. **Data labels:** Show data values directly on bars/lines when there are ≤8 data points. For dense data, omit data labels and rely on axes.
4. **Grid lines:** Remove horizontal grid lines from bar charts. Keep subtle grid lines on line charts only if they aid readability. Never use vertical grid lines.
5. **Legend:** Include only when there are 2+ data series. Position at bottom or right — never on top of the chart area.
6. **Font:** Match the deck's body font. Minimum 10pt for axis labels, 12pt for data labels.
7. **Aspect ratio:** Charts should be wider than tall (roughly 16:9 or 3:2). Never square. Never taller than wide.

### Chart Accuracy Verification

After generating every chart, verify programmatically:

```python
# For every chart, print the exact values being plotted
print(f"Chart: {chart_title}")
print(f"Labels: {labels}")
print(f"Values: {values}")
print(f"Sum check: {sum(values)}")
# Compare against the auditor's Verified Metrics
```

If any chart value doesn't match the verified data, fix it before embedding. Log every verification in a comment in the generation script.

### Waterfall Charts (Matplotlib → PNG)

For EBITDA bridges, budget variances, and revenue walks — the chart type C-levels request most — use matplotlib since pptxgenjs doesn't support native waterfall:

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import numpy as np

def create_waterfall(categories, values, title, colors, filepath, figsize=(10, 5.5)):
    """
    Creates a waterfall chart and saves as high-res PNG.
    categories: list of labels
    values: list of numeric values (positive = increase, negative = decrease)
    colors: dict with keys 'increase', 'decrease', 'total', 'connector'
    """
    cumulative = np.zeros(len(values))
    cumulative[0] = values[0]
    for i in range(1, len(values)):
        if i == len(values) - 1:  # Last bar is total
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

    # Add connector lines
    for i in range(len(values) - 2):
        ax.plot([i - 0.3, i + 1.3], [cumulative[i], cumulative[i]],
                color=colors['connector'], linewidth=0.8, linestyle='--')

    # Data labels
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

Save waterfall PNGs to `/home/claude/charts/` and embed them in slides using `slide.addImage({ path: ... })`.

---

## 5. Visual Design Standards

### If Brand Guidelines Are Provided
Follow them exactly. Brand colors, fonts, and templates override everything below. Match their system — do not "improve" it.

### Default Design System (When No Brand Guidelines Exist)

**Color palette:** Select from the palettes in the pptx skill's Design Ideas section, or create a custom palette that matches the topic. The palette must have:
- 1 dominant color (60-70% visual weight — used for backgrounds, headers)
- 1 supporting color (20-30% — used for secondary elements, chart fills)
- 1 accent color (10% — used sparingly for emphasis, callouts, key numbers)
- White or near-white for content backgrounds
- Dark charcoal (not black) for body text

**Typography:** Select a font pairing from the pptx skill's Typography section. Use the header font for slide titles and large callouts. Use the body font for all other text.

| Element | Size | Weight |
|---------|------|--------|
| Slide title (action title) | 28-36pt | Bold |
| KPI callout number | 60-72pt | Bold |
| KPI label / subtitle | 12-14pt | Regular |
| Body text | 14-16pt | Regular |
| Chart labels | 10-12pt | Regular |
| Source citations | 8-10pt | Italic, muted color |

**Layout rules:**
- 16:9 aspect ratio (LAYOUT_16x9), always
- 0.5" minimum margins on all sides
- Maximum 3 bullet points per slide (if bullets are used at all — prefer visual layouts)
- One core idea per slide
- Every slide has at least one visual element (chart, icon, callout number, or shape)
- Dark background for title + conclusion slides, light background for content slides

**Icons:** Use react-icons (Font Awesome, Material Design, Heroicons) as documented in the pptx skill. Render to PNG at 256px+ and embed. Use icons in colored circles next to section headers or as visual anchors in card layouts.

### The "No Text Walls" Rule

If a slide has more than 50 words of body text, it's wrong. Rewrite it. Options:
- Break into two slides
- Replace text with a chart or visual
- Distill to 3 bullet points of ≤15 words each
- Use a large number callout with a single sentence of context

---

## 6. Executive Language Standards

### Headline Rules
Every slide title is an **action title** — a complete sentence that states the insight or conclusion. The audience should understand the slide's message from the title alone, without reading anything else.

| ❌ Descriptive (never use) | ✅ Action title (always use) |
|---------------------------|----------------------------|
| "Q3 Revenue Overview" | "Q3 Revenue Exceeds Target by 12%, Driven by Enterprise Segment" |
| "Cost Analysis" | "Logistics Consolidation Saves $1.4M Annually" |
| "Customer Metrics" | "Net Retention at 118% — Expansion Revenue Offsets Churn" |
| "EBITDA Summary" | "EBITDA Margin Expanded 340bps YoY to 22.1%" |

### Vocabulary Standards
- Use "material," "strategic," "ROI-positive," "margin-accretive," "capital-efficient"
- Avoid "amazing," "incredible," "game-changing," "cutting-edge," "leverage" (as a verb), "synergy"
- Reframe process as outcome: Not "We analyzed 50K transactions" but "Transaction analysis reveals $800K in recoverable leakage"
- Quantify everything: Not "significant growth" but "34% YoY growth ($2.1M incremental)"

### The "So What" Test
Before finalizing every slide, apply this test:

```
IF an executive looks at this slide and asks "So what?" or "Why should I care?"
AND the answer is not immediately obvious from the title and visual
THEN the slide fails — rewrite the title, add context, or remove the slide
```

This test is not optional. Apply it to every slide. If a slide exists only to "show data" without driving toward the decision identified in Question 2, cut it or rework it.

---

## 7. Execution Workflow

Follow this exact sequence for every deck:

### Phase 1: Intake
1. Receive user request and data files
2. Ask missing briefing questions (Section 2) — only what's missing
3. Confirm understanding of the audience, decision, and key metrics

### Phase 2: Data Verification
4. If raw data provided → run data-auditor skill → wait for Integrity Report
5. If audit is RED → stop, report to user, wait for resolution
6. If audit is GREEN/YELLOW → extract Verified Metrics for use in slides

### Phase 3: Narrative Planning
7. Draft the deck outline: list every slide with its action title and visual type
8. Present the outline to the user: "Here's the narrative arc I'm proposing. Does this structure work, or should I adjust?"
9. Wait for user approval or edits before building

### Phase 4: Build
10. Read the pptx skill (`/mnt/skills/public/pptx/SKILL.md`) and creation guide (`/mnt/skills/public/pptx/pptxgenjs.md`)
11. Generate all charts (pptxgenjs native or matplotlib → PNG) with accuracy verification
12. Build the full deck using pptxgenjs
13. Embed all charts, icons, and visual elements

### Phase 5: QA (Mandatory — Never Skip)
14. Follow the pptx skill's QA protocol exactly:
    - Content QA with `markitdown`
    - Visual QA by converting to images and inspecting every slide
    - Verify every number on every slide against the audit's Verified Metrics
15. Fix issues, re-render, re-inspect (the verification loop)
16. Do not deliver until QA passes with zero issues

### Phase 6: Deliver
17. Save final `.pptx` to `/mnt/user-data/outputs/`
18. Present the file to the user
19. Provide a brief summary: slide count, key narrative arc, and any caveats from the data audit

---

## 8. Edge Cases

| Scenario | Action |
|----------|--------|
| User provides data but no context on audience or decision | Ask briefing questions. Do not guess the audience. |
| Data audit returns RED | Show findings. Ask user to fix data or acknowledge risks before proceeding. |
| User asks for >15 slides | Push back: "Executive decks lose impact beyond 10-12 slides. Can we move [X] to an appendix?" Comply if they insist, but flag it. |
| User wants a specific chart type that doesn't suit the data | Explain why an alternative is better, show both if they insist. |
| Data has no time dimension (no YoY/MoM possible) | Use category comparisons, rankings, or benchmarks instead. State that trend analysis isn't possible with the available data. |
| User provides a template/existing deck | Use the pptx editing workflow instead of creating from scratch. Match the template's style exactly. |
| Multiple audiences for the same data | Create one deck per audience, or clearly section the deck with audience-specific framing. |
| User says "just make it look good, I don't care about the details" | Still ask Question 2 (the decision) — without it, the deck has no spine. Style can be defaulted; purpose cannot. |
| Confidential/sensitive data | Never include raw data rows in slides. Use aggregated metrics only. Note: the appendix can include detail tables if the user requests it. |

---

## 9. What This Skill Does NOT Do

- It does not perform data analysis or exploration (the auditor handles verification; analysis is a separate workflow)
- It does not create training decks, onboarding materials, or internal process documentation
- It does not design logos, brand identities, or marketing collateral
- It does not write the speaker notes or talking points (unless explicitly requested)
- It does not replace the pptx skill — it builds ON TOP of the pptx skill's technical capabilities

This skill's scope is: verified data → narrative structure → visual executive presentation → delivered .pptx file.
