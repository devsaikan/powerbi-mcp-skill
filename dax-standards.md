# Power BI MCP - DAX Authoring Standards

Reference file for SKILL.md. Apply these standards to every measure, calculated
column, and calculated table created during an MCP session.

---

## Measure Header Template

Every measure must include this header block as a DAX comment:

-- Measure:  [Measure Name]
-- Folder:   [Display Folder Path]
-- Format:   [Format String]
-- Purpose:  [One-line business description]
-- Author:   AI-Assisted - review before publishing
-- Created:  [Session Date]

[Measure Name] =
    [DAX expression - CALCULATE and major functions
     on separate lines for readability]

---

## DAX Quality Rules

### Division
Always use DIVIDE - never bare /

Correct:
Margin % = DIVIDE([Gross Margin], [Total Revenue], 0)

Never:
Margin % = [Gross Margin] / [Total Revenue]

### Filter Context
Always use CALCULATE with explicit filter context modification.
Never rely on implicit filter propagation for measures that need isolation.

Revenue LY =
    CALCULATE(
        [Total Revenue],
        SAMEPERIODLASTYEAR('Date'[Date])
    )

### Time Intelligence
Always specify ALL or ALLEXCEPT scope explicitly.
Never leave time intelligence measures ambiguous about what they remove.

Revenue YTD =
    CALCULATE(
        [Total Revenue],
        DATESYTD('Date'[Date])
    )

Revenue YTD (Fiscal) =
    CALCULATE(
        [Total Revenue],
        DATESYTD('Date'[Date], "06/30")
    )

### Matrix / Visual Context
Use ISINSCOPE for dynamic header measures that need to respond to
visual hierarchy level.

Dynamic Label =
    IF(
        ISINSCOPE('Product'[SubCategory]),
        "SubCategory Total",
        "Category Total"
    )

### Slicer-Driven Measures
Use SELECTEDVALUE with a safe default for slicer-controlled logic.

Selected Metric =
    SWITCH(
        SELECTEDVALUE('Metric Selector'[Metric], "Revenue"),
        "Revenue",    [Total Revenue],
        "Margin",     [Gross Margin],
        "Units",      [Units Sold],
        [Total Revenue]
    )

### Performance - Filter Arguments
Prefer filter arguments in CALCULATE over FILTER(ALL(...)) when
filtering on a single column value. It is faster and easier to read.

Correct (faster):
Online Revenue =
    CALCULATE([Total Revenue], 'Sales'[Channel] = "Online")

Avoid (slower, verbose):
Online Revenue =
    CALCULATE(
        [Total Revenue],
        FILTER(ALL('Sales'), 'Sales'[Channel] = "Online")
    )

Use FILTER(ALL(...)) only when filtering on a measure or complex expression.

### Blank Handling
Use IF(ISBLANK(...), 0, ...) intentionally. Do not silently return blanks
in measures that feed visuals unless the blank is meaningful.

---

## Format Strings

Currency (USD):      $#,0.00
Currency (no cents): $#,0
Percentage:          0.00%
Percentage (whole):  0%
Integer:             #,0
Decimal (2dp):       #,0.00
Large numbers:       #,0,,.0"M"
Date:                MMM DD, YYYY

---

## Display Folder Taxonomy

Organize all measures into display folders. Adapt folder names to the model
domain. Use the underscore prefix for folders that should sort to the top.

_Key Metrics       - top-level KPIs visible at a glance
Financial          - revenue, margin, cost, P&L measures
Time Intelligence  - YTD, MTD, QTD, PY, rolling averages
Customer           - acquisition, retention, LTV, churn
Operational        - fulfillment, inventory, SLA, lead time
Marketing          - campaign performance, attribution, ROAS
Product            - SKU-level, category, returns
[Domain-specific]  - add folders as needed for the model
_Debug             - diagnostic measures, hide before publishing

Rules:
- Never leave measures in the root with no folder assigned
- Debug measures must have IsHidden = true before any report publish
- Folder names are case-sensitive in Power BI - be consistent

---

## Naming Conventions

Measure:            Title Case              Revenue YTD
Calculated column:  Title Case              Customer Age Band
Calculated table:   Title Case, _Calc       Date_Calc
Helper measure:     Underscore prefix       _Filter Context Check
Debug measure:      DEBUG_ prefix           DEBUG_Row Count
Parameter table:    Param suffix            Discount Rate Param

---

## Dependency Awareness

Before creating or modifying any measure, check:
- Does this measure reference other measures that may be deleted or renamed?
- Does this measure sit in a calculation chain that will affect existing visuals?
- Does a relationship change break existing measures that depend on cross-filter?

For models with 50 or more measures, run a dependency scan (Tier 1 action) before
any structural modification session begins.

---

## Line and Trend Chart Visual Intelligence

These rules apply when MCP is assisting with measure design that will feed line
or trend chart visuals. Claude should surface these standards proactively when
the user is working on time-series measures.

### Y-Axis Scaling - Semantic Model Responsibility

The semantic model cannot control axis scaling directly (that lives in the report
layer), but measure design can influence what the axis must show.

Do not compress signal with over-broad ranges. If a measure like Gross Margin %
realistically lives in a 35-45% band, document that expected range in the measure
description. This gives the report author context to set a meaningful axis range
rather than defaulting to 0-100%.

Measure description template:

[Gross Margin %]
Business range: 35%-45% (industry-typical)
Use axis min/max bounded to this range in line charts.
Zero-based axis will flatten signal for this metric.

Zero-based axis rule:
- Bar/column charts: always zero-based (bar height encodes magnitude)
- Line charts: zero-based only when the full range is semantically meaningful
  (e.g., a completion % that genuinely spans 0-100%)
- For metrics with a natural operating band (margin %, return rate): bound the
  axis to show the data occupying most of chart height

### Dual Axes - Avoid Unless Pareto

Do not design measure pairs intended for dual-axis line charts.

Dual Y axes imply correlation between two lines that may not exist. The crossing
point is an artifact of axis scale choice, not a meaningful intersection.
Changing either axis range moves or eliminates the crossing.

Exception: Pareto charts only. Bars (ranked revenue/volume) plus a cumulative %
line encode categorically different things and are not meant to be read at their
crossing point.

Preferred alternatives:
1. Vertically aligned single-axis charts with a shared X axis - each measure gets
   its own unambiguous scale
2. Normalize both measures to a common unit (e.g., % change from baseline) so
   they share one axis honestly

Normalization caveat: If one measure is already a percentage, % change from
baseline produces a percentage of a percentage. A 4pp move from 40% to 44% reads
as +10%, which is easy to misread. Flag this in measure descriptions.

### Blended Metrics - Always Question the Mix Effect

When building a blended aggregate measure, always create the breakdown version
alongside it.

A single blended line can hide two things. First, stable sub-groups at different
levels where the blend moves because revenue mix shifts between high-margin and
low-margin segments, not because performance changed. Second, opposing trends
where one sub-group is growing and another is declining, cancelling out into a
falsely calm blend.

Mandatory companion measures for any blended metric:

-- Blended (existing)
Gross Margin % = DIVIDE([Total Gross Margin], [Total Revenue], BLANK())

-- Breakdown (add this)
Gross Margin % by Brand =
CALCULATE(
    DIVIDE([Total Gross Margin], [Total Revenue], BLANK()),
    ALLEXCEPT('Product', 'Product'[Brand])
)

When presenting blended metrics to users, surface the sub-group breakdown option.
The blend is a starting point, not the answer.

### Multi-Series Thresholds

Design measures and recommend visual patterns based on series count.

| Series Count | Recommended Pattern |
|---|---|
| 1-4 | Standard multi-series line chart |
| 5-6 | Visual hierarchy: one strong color, rest to light gray at reduced opacity |
| 7+ | Small multiples - one panel per series, shared axis scale |
| 12+ | Small multiples only; never a single spaghetti chart |

Highlight-and-gray is a narrative move, appropriate for presentations and embedded
briefings where the author already knows which series is the message. In
self-service reports it may be presumptuous and grows outdated as trends change.
Use small multiples for self-service.

Small multiples trade-off: some Power BI features are disabled in small multiples
mode, including trend lines, forecasting, anomaly detection, and X axis label
wrapping. Reference lines survive.

### Interpolation Mode Guidance

When advising on measure granularity and how data will be plotted:

| Scenario | Interpolation Recommendation |
|---|---|
| Aggregated monthly/quarterly data | Monotone smooth - eases corners without overshooting |
| Metrics that hold until they change (prices, rates, thresholds) | Stepped |
| Very low point count (2-3 annual points) | Columns, not a line - a line implies trajectory that is not there |
| Never recommend | Cardinal smooth - can overshoot, producing peaks and valleys not in the data |

When smooth interpolation is used, recommend leaving data point markers visible
so readers can see where observations actually sit, separate from the interpolated
path.

### Time-Intelligence Patterns for Context Layers

Year-over-year overlay (two-line approach):

-- Current year (emphasized)
Total Revenue = [base measure]

-- Prior year (reference)
PY Revenue =
CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date'[Date]))

Plot both on the same axis. Current year emphasized, prior year as reference.
The gap between lines is the YoY variance, readable without a separate chart.

YoY variance (single-line approach):

Revenue YoY Variance =
[Total Revenue] - [PY Revenue]

Use when the variance is the signal and absolute levels matter less. One line
around a zero baseline answers "growing or shrinking?" directly. Reduces ink
budget so it can be spent on other context.

When to use which:
- Absolute levels matter: two-line overlay
- Direction and magnitude of change: single variance line around zero

### Reference and Trend Lines

Maximum one trend line or one reference line per chart. Each additional layer
requires a stated question it answers.

Test before adding: "What question does this layer answer that the reader is
already asking?" If you can articulate it clearly, it might earn its spot. If
you cannot, it does not belong.

Available reference line types in Power BI:
- Constant threshold (e.g., budget target, SLA floor)
- Average, min/max, median, percentile (dynamic)
- Statistical trend line (fitted to data points)
- Forecast and anomaly detection

Note: Multi-series, small multiples, and Legend well usage disables trend lines,
forecasting, and anomaly detection. Only reference lines survive in these
configurations.

### Specialized Chart Patterns

Slope Charts (two-point before/after):
- Use when the question is about change between two specific periods, not the
  full trajectory
- Design: measure at Time A to measure at Time B, per category
- Slope and direction immediately shows who gained, who lost, by how much

Cyclical and Seasonal Data:
- Standard linear time axis puts December and January at opposite ends; seasonal
  wrap-around patterns get flattened
- For "is there a seasonal pattern?" questions, recommend radial or closed-loop
  chart (requires Deneb or custom visual)
- Deneb is the recommended tool when standard Power BI visual constraints are too
  limiting for color encoding or layout control

Pareto Charts (ranked + cumulative %):
- Only legitimate dual-axis use case
- Bars: values sorted by rank (revenue, volume, etc.)
- Line: cumulative % where steep early and flat late means concentration; gradual
  slope means even distribution
- The crossing point of bar and line scales is not meaningful; they encode
  categorically different things
