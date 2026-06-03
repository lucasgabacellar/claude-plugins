---
name: query-assumptions
description: >
  Query and search existing assumptions in the Premissas_Decarb workbook.
  Use whenever the user asks "what assumptions do we have for", "qual premissa",
  "buscar premissa", "search assumptions", "find assumption", "do we have data on",
  "temos premissa de", "show me the assumption for", "quanto é o", "what's the value of",
  or any question about data stored in the assumptions database. Also trigger when the
  user asks about sources, references, or wants to know what's in a specific topic sheet.
  IMPORTANT: answers are confined to data in the workbook tables — never invent or
  guess values that are not in the database.
---

# Query Assumptions

Search and display assumptions from the Premissas_Decarb workbook. All answers are strictly confined to data in the workbook tables.

## Before Starting

Read `references/workbook-structure.md` from the `extract-assumptions` skill for sheet names, column layouts, and table structures.

## Critical Rules

1. **Only report data that exists in the workbook.** Never invent, estimate, or guess values.
2. **If a queried assumption does not exist**, clearly state: "Essa premissa não está cadastrada no banco de dados."
3. **Do not modify the workbook.** This skill is read-only.
4. **Show source information** when available — always include the Source ID and link if present.

## How to Search

### Step 1: Read the Workbook

Use Python with openpyxl to read the workbook:

```python
import openpyxl
wb = openpyxl.load_workbook('path/to/Premissas_Decarb.xlsm', 
                             data_only=True, read_only=True)
```

Use `data_only=True` to get calculated values instead of formulas.

### Step 2: Search Strategy

For a user query, search across ALL 8 topic sheets + Sources table:

1. **Tokenize** the query into keywords
2. **Search** each table row: check columns Assumption (2), Value (3), Unit (4), Source ID (5), Observations (9)
3. **Match** using case-insensitive substring matching. All keywords must match (AND logic) across any combination of columns.
4. **Rank** results by relevance (more keyword matches in the Assumption name = higher rank)

### Step 3: Present Results

Use `show_widget` to display results in a clean card format:

```
┌─────────────────────────────────────────────┐
│ [H-015]  CAPEX Eletrolisador PEM           │
│ Sheet: Hydrogen                             │
│                                             │
│ Value: 1,500          Unit: USD/kW          │
│ Source: IEA-2024-001                        │
│ Access Date: 15/03/2024                     │
│ Responsible: Lucas Bacellar                 │
│ Internal: No                                │
│ Observations: Base case, utility-scale      │
└─────────────────────────────────────────────┘
```

If the source ID is present, also look up the source details from `tblSources` and include:
- Title, Author, Publication Date, Link

### Step 4: Handle No Results

If no results are found:

> "Não encontrei premissas correspondentes a '{query}' no banco de dados Premissas_Decarb."
> 
> Sheets consultadas: Conversions, Hydrogen, Steel, CCUS, Carbon Market, Biogas_Biomethane, Electric Sector, Economic
> 
> Total de premissas no banco: {N}

Optionally suggest related assumptions if any partial matches exist.

## Query Types

### By topic
"What hydrogen assumptions do we have?" → Search Hydrogen sheet, list all

### By keyword
"CAPEX electrolyzer" → Search all sheets for rows containing both "CAPEX" and "electrolyzer"

### By source
"What did we get from IEA?" → Search Source ID column for "IEA", list all matching assumptions

### By sheet
"Show me everything in Carbon Market" → List all assumptions in the Carbon Market sheet

### Summary statistics
"How many assumptions do we have?" → Count rows per sheet, show totals

## Bilingual Support

The workbook supports PT and EN. Search should match keywords in either language. Column headers may be in PT or EN depending on the current language setting, but the DATA content stays in its original language.
