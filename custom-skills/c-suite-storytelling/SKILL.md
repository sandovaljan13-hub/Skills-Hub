---
name: c-suite-storytelling
description: "Use this skill whenever the user wants to create a data-driven executive presentation, board deck, strategy deck, or decision deck for C-level audiences. Trigger when the user says things like 'build me a board deck', 'make a presentation for the CFO', 'create an executive summary deck', 'turn this data into a strategy presentation', 'I need slides for leadership', or provides data (CSV, XLSX) and wants it turned into a persuasive visual narrative. Also trigger when the user mentions 'decision deck', 'EBITDA bridge', 'board meeting', 'investor deck', 'QBR deck', or 'executive review'. This skill chains with the data-auditor skill — if raw data is provided, audit every number before any slide is built. Do NOT trigger for informal presentations, training decks, or presentations that do not target senior leadership audiences."
---

# C-Suite Data Storytelling Engine

You build board-ready presentations that transform verified data into persuasive visual narratives for C-level executives.

The single most important principle: **every slide must answer "So what?"** Executives don't need to see data — they need to see what the data means for a decision they have to make. If a slide shows a number without making clear why the audience should care, that slide is wasting their time. This principle should guide every choice you make — from which metrics to feature, to how you title slides, to whether a chart or a callout number is the right visual.

Before doing anything, read the pptx skill at `/mnt/skills/public/pptx/SKILL.md` and its creation guide at `/mnt/skills/public/pptx/pptxgenjs.md`. Those contain the technical foundation for building slides. This skill governs the *strategic and narrative layer* on top of that foundation.

For chart generation details, anchoring techniques, and visual examples, read `references/charts-and-visuals.md`.
For the executive anchoring and persuasion framework, read `references/anchoring-framework.md`.

---

## Data Integrity: Auditor Before Storyteller

Every number on every slide needs to be verified. The reason is simple — if a board member catches one wrong number, the entire deck loses credibility, and so does the presenter.

When the user provides raw data (CSV, XLSX, Parquet, JSON), run the data-auditor skill first. Wait for the Data Integrity Report. If the audit status is 🔴 RED, stop and show the user the findings — building slides on bad data is worse than building no slides at all. If GREEN or YELLOW, extract the Verified Metrics table and use it as the single source of truth for every number that appears on a slide.

If the user says the data is pre-verified, skip the audit — but still compute any derived metrics (YoY changes, percentages, variances) with code. Mental math and eyeballing have no place in an executive deck.

---

## The Briefing: Understanding Before Building

Before generating slides, you need clarity on five things. The reason you ask these isn't process for process's sake — it's because a deck for a CFO making a budget decision looks fundamentally different from a deck for a Board reviewing strategic options, even if the underlying data is identical.

1. **Who is the audience?** (CFO, Board, CEO, Sales VPs, Investors)
2. **What decision should they make after this deck?** — this is the spine of the entire narrative
3. **What is the single most important takeaway?** — this becomes your Executive Summary slide
4. **Which specific metrics/KPIs matter?** (Revenue, EBITDA, CAC, churn, margins, etc.)
5. **Are there brand guidelines?** (colors, fonts, logo, template)

If the user's prompt already answers some of these, don't re-ask — confirm your understanding and ask only what's missing. If the user gives a fully detailed brief, confirm in 2-3 sentences and get to work. The goal is to gather what you need without creating a wall of questions.

The one question you should always get an answer to, even if the user says "just make it look good," is #2 — the decision. Without it, the deck has no direction. Everything else can be defaulted.

---

## Narrative Architecture

Every deck follows this arc. The reason for this structure is that C-level audiences are time-constrained and decision-oriented — they want the conclusion first, the evidence second, and the options third. This is sometimes called "pyramid principle" or "BLUF" (Bottom Line Up Front).

**Title Slide** → **Executive Summary** → **Burning Platform** → **Evidence** → **Options** → **Recommendation + Inaction Cost** → **Risk & Mitigation** → **Appendix (if needed)**

Key principles for each:

**Title Slide:** The title is an action title, not a label. "Q3 Financial Review" tells the audience nothing. "Q3 Momentum: $2.1M EBITDA Beat Creates Runway for Phase 2 Expansion" tells them the story before they've even turned the page.

**Executive Summary:** The most important slide. If the CEO leaves after 60 seconds, this slide stands alone. Use 3-4 large stat callouts (60-72pt numbers) with a single narrative sentence and the explicit ask. No bullets, no paragraphs, no anchoring devices — just the core numbers and what you want them to do.

**Burning Platform:** Why act now? What happens if we don't? This is where urgency lives, and where the anchoring framework (see `references/anchoring-framework.md`) is most powerful. Use inaction framing and scale comparisons to make the cost of delay feel real.

**Evidence Slides:** One insight per slide. Each slide title states the conclusion, not the topic — the audience should understand the message from the title alone without reading anything else. Each slide gets one chart or visual and one anchoring device that translates the key metric into something visceral.

**Options / Comparison:** Present 2-3 strategic paths with quantified trade-offs. Use a card or column layout so the audience can compare side by side. The contrast between options does most of the persuading here — keep anchoring light.

**Recommendation:** Clear, singular recommendation. Quantified expected outcome. Explicit next steps with owners and dates. And critically — include the cost of inaction. Executives are loss-averse; "here's what we gain" is less persuasive than "here's what we lose every quarter we delay."

**Risk & Mitigation:** Top 3 risks, each with a concrete mitigation. Pre-empts the "what could go wrong?" question that C-levels will always ask. Clean table or card layout.

**Appendix:** Methodology, detailed data tables, supporting analysis. Clearly labeled. Process and methodology live here, never in the main deck — executives don't care how you got the numbers, they care what the numbers mean.

---

## Execution Workflow

1. **Intake** — receive request and data, ask only the missing briefing questions, confirm understanding
2. **Audit** — if raw data provided, run data-auditor, wait for GREEN/YELLOW before proceeding
3. **Plan** — draft the deck outline with every slide's action title, visual type, and planned anchoring device. Present the outline to the user for approval before building. This step matters because changing a slide's framing after it's built is 10x more expensive than adjusting an outline.
4. **Build** — read the pptx skill, generate all charts with accuracy verification (see `references/charts-and-visuals.md`), compute all anchoring comparisons with code, build the deck
5. **QA** — follow the pptx skill's QA protocol: content QA with `markitdown`, visual QA by converting to images and inspecting every slide, verify every number against the audit's Verified Metrics, verify every anchoring calculation. Fix and re-verify until clean. This is not optional — the first render almost always has issues.
6. **Deliver** — save to `/mnt/user-data/outputs/`, present the file, provide a brief summary

---

## Slide Design Principles

Read the pptx skill's Design Ideas section for color palettes, font pairings, and layout patterns — those are your technical toolkit. Here's how to apply them for executive audiences specifically:

**Every slide title is an action title.** Not "Revenue by Region" but "APAC Revenue Up 34% — Now Our Fastest-Growing Market." The audience should never have to read the slide body to understand the message. The reason this matters is that many executives skim — they flip through the deck reading only titles. If your titles are labels, the deck is incomprehensible at skim speed.

**One idea per slide, one visual per slide.** Executive attention is finite. When a slide tries to make two points, neither lands. When in doubt, split into two slides rather than cramming.

**Keep text minimal.** If a slide has more than ~50 words of body text, it's probably trying to do too much. Break it into two slides, replace text with a chart, or distill to a few key phrases. The reason: dense text signals "this is a document pretending to be a presentation," which executives have been trained to ignore.

**Dark backgrounds for bookend slides (title + conclusion), light for content.** This creates visual rhythm and signals to the audience where they are in the narrative arc.

**No accent lines under titles.** The pptx skill flags this specifically as a hallmark of AI-generated slides. Use whitespace or background color to create visual hierarchy instead.

---

## Language Standards

**Action titles, always.** See the narrative architecture section above.

**Vocabulary:** Use "material," "strategic," "ROI-positive," "margin-accretive," "capital-efficient." Avoid "amazing," "game-changing," "cutting-edge," "leverage" (as a verb), "synergy." These words signal fluff, not substance. Reframe process as outcome: not "We analyzed 50K transactions" but "Transaction analysis reveals $800K in recoverable leakage."

**Quantify everything.** "Significant growth" means nothing. "34% YoY growth ($2.1M incremental)" is a fact. Executives trust specificity.

---

## Edge Cases

| Scenario | Action |
|----------|--------|
| User provides data but no context on audience or decision | Ask briefing questions — at minimum, get the decision question answered |
| Data audit returns RED | Show findings, ask user to resolve before proceeding |
| User wants >15 slides | Push back — executive decks lose impact beyond 10-12 slides, suggest moving detail to appendix. Comply if they insist but flag it. |
| User wants a specific chart type that doesn't fit | Explain why an alternative works better, offer to show both |
| No time dimension in data | Use category comparisons or rankings instead, note that trend analysis isn't possible |
| User provides existing template | Use the pptx editing workflow, match template style exactly |
| User says "just make it look good" | Still get the decision question answered — style can be defaulted, purpose cannot |

---

## What This Skill Does NOT Do

This skill's scope is: verified data → narrative structure → visual executive presentation → delivered .pptx file.

It does not perform exploratory data analysis, create training materials, design brand identities, or write speaker notes (unless explicitly asked). It does not replace the pptx skill — it builds on top of the pptx skill's technical capabilities.
