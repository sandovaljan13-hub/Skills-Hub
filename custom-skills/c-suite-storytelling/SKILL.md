---
name: c-suite-storytelling
description: "Use this skill whenever the user wants to create a data-driven executive presentation, board deck, strategy deck, or decision deck for C-level audiences. Trigger when the user says things like 'build me a board deck', 'make a presentation for the CFO', 'create an executive summary deck', 'turn this data into a strategy presentation', 'I need slides for leadership', or provides data (CSV, XLSX) and wants it turned into a persuasive visual narrative. Also trigger when the user mentions 'decision deck', 'EBITDA bridge', 'board meeting', 'investor deck', 'QBR deck', or 'executive review'. This skill chains with the data-auditor skill — if raw data is provided, audit every number before any slide is built. Do NOT trigger for informal presentations, training decks, or presentations that do not target senior leadership audiences."
---

# C-Suite Decision Engine — Hybrid Minto/SCQA Methodology

You build board-ready presentations that **drive executive decisions**, not just present data. Every deck is structured around a single decision the audience must make, argued through the SCQA (Situation → Complication → Question → Answer) logic flow.

The single most important principle: **every slide must drive toward a decision.** Executives don't attend presentations to be informed — they attend to decide. If a slide doesn't advance the argument toward the recommended action, cut it. If a title describes a topic instead of stating a financial implication, rewrite it. The deck is a persuasion instrument, not a reporting artifact.

Before doing anything, read the pptx skill at `/mnt/skills/public/pptx/SKILL.md` and its creation guide at `/mnt/skills/public/pptx/pptxgenjs.md`. Those contain the technical foundation for building slides. This skill governs the *strategic, narrative, and decision-logic layer* on top of that foundation.

For chart generation details and visual examples, read `references/charts-and-visuals.md`.
For the executive anchoring, logic bridging, and persuasion framework, read `references/anchoring-framework.md`.

---

## Data Integrity: Auditor Before Storyteller

Every number on every slide needs to be verified. The reason is simple — if a board member catches one wrong number, the entire deck loses credibility, and so does the presenter.

When the user provides raw data (CSV, XLSX, Parquet, JSON), run the data-auditor skill first. Wait for the Data Integrity Report. If the audit status is 🔴 RED, stop and show the user the findings — building slides on bad data is worse than building no slides at all. If GREEN or YELLOW, extract the Verified Metrics table and use it as the single source of truth for every number that appears on a slide.

If the user says the data is pre-verified, skip the audit — but still compute any derived metrics (YoY changes, percentages, variances) with code. Mental math and eyeballing have no place in an executive deck.

---

## The Briefing: SCQA Discovery

Before generating slides, you need to build the SCQA backbone. Without it, the deck has no argumentative spine. The briefing extracts six things — three structural (Situation, Complication, Skeptic) and three tactical (Decision, Metrics, Brand).

### Structural Discovery (always required)

1. **Situation** — What is the current state the audience already accepts as true? This is the common ground you establish before introducing tension. It should be factual, uncontroversial, and brief. Example: "We expanded into APAC last year with a $5M investment and achieved $3.2M in first-year revenue."

2. **Complication** — What has changed, gone wrong, or emerged that creates tension against the Situation? This is the "but" or "however" that makes the status quo untenable. It can be a threat (competitive entry, margin erosion, regulatory change), a missed target, or an emerging opportunity with a closing window. Example: "But two direct competitors entered the same markets in Q2, and our customer acquisition cost has risen 40% in six months."

3. **Skeptic** — Who in the room will resist the recommendation, and why? This is audience analysis with teeth. Identify the specific objection the hardest-to-convince person will raise, because the deck must preemptively neutralize it. Example: "The VP of Operations will push back because the last expansion strained delivery capacity and they're still catching up."

### Tactical Discovery (ask only what's missing)

4. **What decision should they make after this deck?** — This is the Answer in SCQA. It becomes the title slide and the recommendation. Without it, the deck has no direction. Everything else can be defaulted; this cannot.

5. **Which specific metrics/KPIs matter?** (Revenue, EBITDA, CAC, churn, margins, NPS, etc.)

6. **Are there brand guidelines?** (colors, fonts, logo, template)

If the user's prompt already answers some of these, don't re-ask — confirm your understanding and ask only what's missing. If the user gives a fully detailed brief, confirm in 2-3 sentences and get to work.

The one thing you must always extract, even if the user says "just make it look good," is the **Complication**. Without tension, there is no argument. Without an argument, there is no decision. A deck without a Complication is a status update, not a strategy presentation.

---

## Narrative Architecture: SCQA Decision Flow

Every deck follows this arc. The structure is derived from the Minto Pyramid Principle adapted for C-level time constraints: lead with the Answer, then prove it.

**Title Slide (The Answer)** → **SCQA Executive Summary** → **Burning Platform** → **Evidence (Rule of Three)** → **Recommendation + Cost of Inaction**

Optional additions after Recommendation: **Risk & Mitigation** → **Appendix (if needed)**

### Title Slide — The Answer Up Front

The title slide states the recommendation as a fait accompli with its financial implication. This is the Answer in SCQA — the conclusion the entire deck argues toward.

Not "Q3 Financial Review." Not even "Q3 Momentum Creates Expansion Opportunity." Instead: **"Accelerate APAC Phase 2: $8.2M Revenue Opportunity at 24% Margin, Shrinking $800K/Month Without Action."**

The title is the thesis. If the CEO reads nothing else, they know what you want and what it's worth. The financial implication is mandatory — every title must contain a dollar figure, percentage, or quantified outcome.

### SCQA Executive Summary — The Logic in 60 Seconds

The most important slide. If the executive leaves after one minute, this slide must stand alone as a complete argument.

Structure it explicitly as SCQA:
- **S (Situation):** One sentence establishing shared context. "We invested $5M in APAC and achieved $3.2M first-year revenue, validating market demand."
- **C (Complication):** One sentence introducing the tension. "Two competitors entered in Q2; our CAC rose 40% and the addressable whitespace is shrinking $800K/month."
- **Q (Question):** The implicit strategic question this creates. "Do we accelerate to capture position, hold steady, or exit?"
- **A (Answer):** The recommendation with quantified outcome. "Accelerate Phase 2 with $2M investment → projected $8.2M revenue within 18 months."

Below the SCQA flow, include 3 large stat callouts (60-72pt numbers) — the three strongest proof points that support the Answer. No bullets, no paragraphs, no anchoring devices on this slide — just the logic flow and the core numbers.

### Burning Platform — Why Now, Not Next Quarter

Why act now? What happens if we don't? This is where urgency lives, where the Complication gets its full weight.

Use inaction framing and scale comparisons from `references/anchoring-framework.md` to make the cost of delay feel real. The Burning Platform must answer one question: **"What do we lose every week/month/quarter we delay this decision?"**

This slide directly addresses the Skeptic identified in the briefing. If the Skeptic's objection is operational capacity, the Burning Platform shows what inaction costs in terms they care about. If the Skeptic is the CFO, use the CFO Inaction Framework from `references/anchoring-framework.md`. Preempt the objection, don't wait for it.

### Evidence — Rule of Three

Present exactly three evidence slides — no more, no fewer. Three is the maximum number of arguments an executive will retain from a single presentation. Each of the three slides proves one leg of the argument supporting the Answer.

**Rules for evidence slides:**

1. **One insight per slide, one visual per slide.** Executive attention is finite. When a slide tries to make two points, neither lands.

2. **Every title is a Financial Action Title.** Not a topic, not even an insight — a financial implication. The audience should know the dollar impact from the title alone:
   - Wrong: "Customer Acquisition Costs by Region"
   - Wrong: "APAC CAC Is Rising Faster Than Other Regions"
   - Right: **"Rising APAC CAC Erodes $1.2M in Annual Margin — Requires Channel Mix Shift to Restore Unit Economics"**

3. **Every metric must be logic-bridged.** No metric appears without being translated into its financial consequence (see `references/anchoring-framework.md` Logic Bridging section). NPS doesn't stand alone — it becomes churn risk, which becomes dollar loss. Utilization rate doesn't stand alone — it becomes idle capacity cost.

4. **Each slide gets one chart or visual and one anchoring device** that translates the key metric into something visceral.

### Recommendation + Cost of Inaction

Clear, singular recommendation. This slide has two halves that work as a contrast:

**Left/Top: The Recommendation**
- Quantified expected outcome with timeline
- Explicit next steps with owners and dates
- Investment required and expected return

**Right/Bottom: Cost of Inaction**
- What happens each quarter if we don't act, expressed in dollars lost, market position degraded, or operational risk increased
- Keep this accessible to any executive — use plain business language, not finance jargon
- When the Skeptic or primary audience is a CFO, escalate to the CFO Inaction Framework from `references/anchoring-framework.md` (payback periods, capital efficiency, ROI erosion per quarter)

Executives are loss-averse. "Here's what we gain" is less persuasive than "here's what we lose every quarter we delay." Always include both.

### Risk & Mitigation (optional but recommended)

Top 3 risks, each with a concrete mitigation. Pre-empts the "what could go wrong?" question. Clean table or card layout. Include this slide when the investment or change is significant enough that risk questions are inevitable.

### Appendix (if needed)

Methodology, detailed data tables, supporting analysis. Clearly labeled. Process and methodology live here, never in the main deck — executives don't care how you got the numbers, they care what the numbers mean.

---

## Financial Action Titles — The Non-Negotiable Standard

Every slide title in the deck must be a **Financial Action Title** — a sentence that states a financial implication and implies or states the required action. This is not a suggestion; it is the single most important formatting rule in this methodology.

The test: cover the slide body. If the title alone tells the executive the financial impact and what to do about it, it passes. If the title is a topic label or a data description, it fails.

| Fails (topic label) | Fails (insight without $) | Passes (Financial Action Title) |
|---------------------|--------------------------|-------------------------------|
| "Revenue by Region" | "North Region Leads in Growth" | "North's 34% Growth Delivers $2.1M Incremental EBITDA — Double Down on Channel Strategy" |
| "Customer Churn Analysis" | "Churn Rate Exceeds Industry Average" | "3-Point Churn Gap Costs $4.2M Annually — Retention Program Pays Back in 2 Quarters" |
| "Operational Efficiency" | "Utilization Rates Are Below Target" | "45% Utilization Wastes $1.8M in Idle Capacity — Automation Recovers 60% Within 6 Months" |
| "NPS Trends" | "NPS Declined 12 Points This Year" | "NPS Decline Signals 8% Incremental Churn Risk — $6.4M Revenue at Stake Without Intervention" |

---

## Execution Workflow

1. **Intake** — receive request and data, run SCQA Discovery to extract Situation, Complication, Skeptic, and Decision. Confirm understanding.
2. **Audit** — if raw data provided, run data-auditor, wait for GREEN/YELLOW before proceeding
3. **Plan** — draft the deck outline with every slide's Financial Action Title, SCQA position (which part of the argument it serves), visual type, and planned anchoring device. Present the outline to the user for approval before building. This step matters because changing a slide's framing after it's built is 10x more expensive than adjusting an outline.
4. **Build** — read the pptx skill, generate all charts with accuracy verification (see `references/charts-and-visuals.md`), compute all logic bridges and anchoring comparisons with code, build the deck
5. **QA** — follow the pptx skill's QA protocol: content QA with `markitdown`, visual QA by converting to images and inspecting every slide, verify every number against the audit's Verified Metrics, verify every anchoring calculation and logic bridge. Fix and re-verify until clean. This is not optional — the first render almost always has issues.
6. **Deliver** — save to `/mnt/user-data/outputs/`, present the file, provide a brief summary of the SCQA argument the deck makes

---

## Slide Design Principles

Read the pptx skill's Design Ideas section for color palettes, font pairings, and layout patterns — those are your technical toolkit. Here's how to apply them for executive decision decks specifically:

**Every slide title is a Financial Action Title.** See the dedicated section above. The audience should never have to read the slide body to understand the financial implication. Many executives skim — they flip through reading only titles. If your titles are topic labels, the deck is incomprehensible at skim speed. If your titles state financial implications, the deck's argument is complete from titles alone.

**One idea per slide, one visual per slide.** When in doubt, split into two slides rather than cramming.

**Keep text minimal.** If a slide has more than ~50 words of body text, it's trying to do too much. Break it into two slides, replace text with a chart, or distill to a few key phrases.

**Dark backgrounds for bookend slides (title + recommendation), light for content.** This creates visual rhythm and signals to the audience where they are in the narrative arc.

**No accent lines under titles.** The pptx skill flags this specifically as a hallmark of AI-generated slides. Use whitespace or background color to create visual hierarchy instead.

---

## Language Standards

**Financial Action Titles, always.** See the dedicated section above.

**Vocabulary:** Use "material," "strategic," "ROI-positive," "margin-accretive," "capital-efficient." Avoid "amazing," "game-changing," "cutting-edge," "leverage" (as a verb), "synergy." These words signal fluff, not substance. Reframe process as outcome: not "We analyzed 50K transactions" but "Transaction analysis reveals $800K in recoverable leakage — equivalent to funding Q4 hiring without incremental budget."

**Quantify everything.** "Significant growth" means nothing. "34% YoY growth ($2.1M incremental EBITDA)" is a fact. Executives trust specificity.

**Every number earns its place through a logic bridge.** A metric that isn't connected to a financial consequence is trivia. See `references/anchoring-framework.md` for the Logic Bridging framework.

---

## Edge Cases

| Scenario | Action |
|----------|--------|
| User provides data but no context on audience or decision | Run SCQA Discovery — at minimum, extract the Complication and Decision |
| Data audit returns RED | Show findings, ask user to resolve before proceeding |
| User can't articulate the Complication | Help them find it — ask "What's changed?" or "What goes wrong if we do nothing?" The Complication is always there; sometimes it just needs to be surfaced |
| User wants >10 slides | Push back — SCQA decision decks lose impact beyond 8-10 slides (Title + SCQA Summary + Burning Platform + 3 Evidence + Recommendation + Risk). Suggest moving detail to appendix. Comply if they insist but flag it. |
| User wants a specific chart type that doesn't fit | Explain why an alternative works better, offer to show both |
| No time dimension in data | Use category comparisons or rankings instead, note that trend analysis isn't possible |
| User provides existing template | Use the pptx editing workflow, match template style exactly |
| User says "just make it look good" | Still extract the Complication — style can be defaulted, the argument cannot |
| Skeptic is unknown | Default to a general "why now?" objection — frame the Burning Platform around competitive timing or operational cost of delay. Escalate to CFO-specific framing only if the audience is explicitly finance-oriented. |

---

## What This Skill Does NOT Do

This skill's scope is: verified data → SCQA argument structure → Financial Action Titles → visual executive presentation → delivered .pptx file.

It does not perform exploratory data analysis, create training materials, design brand identities, or write speaker notes (unless explicitly asked). It does not replace the pptx skill — it builds on top of the pptx skill's technical capabilities with a decision-driving strategic layer.
