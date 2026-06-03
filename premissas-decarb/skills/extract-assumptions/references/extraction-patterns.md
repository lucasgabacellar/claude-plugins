# Extraction Patterns

How to identify and extract assumptions from different document types.

## General Principles

### Always Find the Primary Source

Documents often compile data from other sources. A dashboard may cite "PSR" but the actual data comes from IEA, VERRA, World Bank, etc. Always look for:

- Footnotes, endnotes, or bibliography sections
- Small-print source citations near tables/charts (e.g., "Source: IEA WEO 2024")
- Reference sheets/tabs in Excel files (e.g., "Referências", "References", "Bibliografia")
- Inline citations like "(Author, Year)" or "[n]" with a numbered reference list
- URLs embedded in cells or text

The primary source is the ORIGINAL publisher of the data, not the intermediary who compiled it.

### What Constitutes an Assumption

An assumption is a quantitative or qualitative premise used in analysis:

- Has a **name/description** (what it represents)
- Has a **value** (numeric, range, or categorical)
- Has a **unit** (currency, percentage, physical unit, or "dimensionless")
- Has a **source** (where it came from)
- May have **observations** (context, caveats, methodology notes)

NOT assumptions: calculated results, intermediate formulas, formatting, labels without values.

### Sheet Assignment Heuristics

When suggesting which topic sheet an assumption belongs to:

| Keywords/Topics | Suggested Sheet |
|---|---|
| Conversion factors, units, energy equivalents, LHV, HHV, density | Conversions |
| Hydrogen, H₂, electrolysis, electrolyzer, fuel cell, CAPEX/OPEX H2 | Hydrogen |
| Steel, DRI, blast furnace, BOF, EAF, iron ore, green steel | Steel |
| CO₂ capture, CCS, CCUS, DAC, storage, sequestration, pipeline CO₂ | CCUS |
| Carbon credit, carbon price, ETS, offset, REDD+, VERRA, Gold Standard | Carbon Market |
| Biogas, biomethane, anaerobic digestion, landfill gas, biomass gasification | Biogas_Biomethane |
| LCOE, solar, wind, hydro, power plant, capacity factor, tariff, grid, TUST | Electric Sector |
| GDP, inflation, exchange rate, discount rate, WACC, SELIC, IPCA, macroeconomic | Economic |

Always present the suggestion to the user for confirmation. If ambiguous, ask.

---

## Pattern 1: Flat Table (e.g., premissas_cocal.xlsx)

**Structure**: Single sheet with columns like #, Category, Description, Value, Unit, Source, Observations.

**Extraction**:
1. Read the table headers to identify column mapping
2. Each row = one assumption
3. Map columns directly: Description → Assumption, Value → Value, Unit → Unit, Source → Source, Observations → Observations
4. The "Category" column helps suggest the target sheet

**Source handling**: Usually a single source name (e.g., "Cocal"). Create one Source entry and reference it for all assumptions from this file.

---

## Pattern 2: Horizontal Blocks (e.g., Dashboard_CCUS, Dashboard_Drivers)

**Structure**: Assumptions organized in side-by-side thematic blocks (Technical, Commercial, Financial). Each block has Label | Value | Unit arranged vertically within the block. Temporal projections may extend horizontally.

**Extraction**:
1. Identify block boundaries (look for section headers like "Premissas Técnicas", "Technical Assumptions")
2. Within each block, find the Label-Value-Unit triplets
3. For temporal data, extract the BASE YEAR value (typically the first year) and note the projection range in Observations
4. Look for reference markers like `[n]` that point to a References sheet/tab
5. Cross-reference `[n]` markers with the References sheet to find the actual source

**Source handling**: These files often have a dedicated References tab with numbered entries. Map each `[n]` to the corresponding reference. Create Source entries for each unique reference found.

**Important**: Distinguish between editable assumptions (user inputs) and calculated values (formulas). Only extract editable assumptions. Look for visual cues: different colors, labels like "Editável" vs "Calculado".

---

## Pattern 3: Matrix Format (e.g., LCOE model)

**Structure**: Rows = parameters, Columns = projects/technologies. Values at intersections. Often with a Unit column.

**Extraction**:
1. Identify the parameter column (usually col B or C) and the unit column
2. Each project column represents a different technology (Solar, Wind, Hydro, etc.)
3. Each cell at (parameter, project) = one assumption
4. Name format: "{Parameter} — {Technology}" (e.g., "CAPEX — Solar Fotovoltaico")
5. Or if the parameter is generic enough, just use the parameter name with the technology in Observations

**Source handling**: Matrix models typically don't track sources per-cell. Ask the user to identify the overall source for the model, or extract from documentation/README tabs if available.

**Temporal handling**: If the matrix has year columns, extract the base-case value and note the projection methodology in Observations.

---

## Pattern 4: Text Document (PDF, Word)

**Structure**: Narrative text with quantitative data embedded in paragraphs, tables, or charts.

**Extraction**:
1. Read the full document text
2. Identify quantitative statements: numbers with units, percentages, costs, dates
3. Look for context: what does the number represent? What are the conditions/assumptions?
4. Extract: description of what the number represents, the value, the unit
5. Find the primary source: check if the document cites another source for each data point

**Source handling**: 
- If the document cites external sources (e.g., "according to IEA (2024)"), use the cited source as the primary source
- If the data is original to the document, use the document itself as the source
- Check for bibliography/references sections at the end

**Important**: Be conservative. Only extract clear, unambiguous quantitative assumptions. Skip vague statements, opinions, or projections without clear methodology.

---

## Pattern 5: Direct Chat Input

**Structure**: User provides assumptions directly in conversation text.

**Extraction**:
1. Parse the user's message for structured data
2. Ask clarifying questions if value, unit, or source is unclear
3. Confirm each assumption before adding

**Source handling**: Ask the user for the source. If they say "internal", mark as Internal = Yes with source = "PSR (internal)".
