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

Currency (USD):      \$#,0.00
Currency (no cents): \$#,0
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
