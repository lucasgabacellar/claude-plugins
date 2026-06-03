---
name: extract-assumptions
description: >
  Extract assumptions from documents and insert them into the Premissas_Decarb workbook.
  Use whenever the user wants to "extract assumptions", "extrair premissas", "add assumptions
  from a document", "incorporar premissas", "import assumptions", "ler premissas de",
  "pegar premissas de", provides a PDF/Word/Excel file containing assumptions, or pastes
  assumption data directly in chat. Also use when the user says "new assumption", "nova
  premissa", or wants to add data to the assumptions database manually. Always use this
  skill even if the user just drops a file without explicit instructions — if the file
  looks like it contains quantitative assumptions, trigger this skill.
---

# Extract Assumptions

Extract quantitative assumptions from documents (PDF, Word, Excel) or chat input, validate each one with the user, and write a JSON import file that the user loads into Excel via VBA.

## Before Starting

Read these reference files:
- `references/safety-rules.md` — **READ THIS FIRST.** Contains the golden rule: NEVER write to the .xlsm file directly.
- `references/workbook-structure.md` — sheet names, column layouts, ID formats
- `references/extraction-patterns.md` — how to parse different document types

## Critical Rules

1. **NEVER open the .xlsm for writing.** openpyxl DESTROYS shapes. Only read with `read_only=True`.
2. **NEVER write to column 1 (IDs).** VBA auto-generates them.
3. **One assumption = one value = one row.** ALWAYS. Never combine multiple data points.
4. **Value field = numeric only.** Decimal separator is "." (dot). No text, ranges, or units.
5. **Range values → compute average.** Note original range in Observations.
6. **Always trace the PRIMARY source** — not the intermediary.
7. **Every item must be validated by the user** — never auto-approve anything.
8. **Observations that contain quantitative data → separate assumption.**

## Workflow

### Step 0: Identify the User

Check `user.name` or `user.email` from the environment. If identification fails, ask once: "Quem está usando? (nome para o campo Responsável)". Store for the session.

### Step 1: Receive Input

Accept: PDF, .docx, .xlsx, .xlsm, .csv, or chat text.
For files, use Read tool or Python to extract content. Select extraction pattern from `references/extraction-patterns.md`.

### Step 2: Extract Assumptions

For each assumption found, extract:

| Field | Required | Rules |
|---|---|---|
| Assumption | YES | One descriptive name. ONE data point per row. |
| Value | YES | Numeric ONLY. Dot decimal. No text/ranges. |
| Unit | YES | Unit of measurement |
| Source | YES | PRIMARY source author + title + year + link |
| Target Sheet | YES | Suggest based on content, user confirms |
| Observations | IF RELEVANT | Context, caveats. NOT quantitative data. |
| Access Date | AUTO | Today's date |
| Responsible | AUTO | Current user |
| Internal | AUTO | "No" for external, "Yes" for PSR-internal |

**Splitting rules:**
- "Volvo 300 km range, Scania 250 km range" → TWO assumptions
- "CAPEX 2025: 1500, 2030: 1200" → TWO assumptions (or one with projection in observations)
- "0.8 to 1.1 km/m³" → Value: 0.95, Observations: "Average of range 0.8-1.1"
- Observation "Consumo médio de X" → This IS an assumption, extract as separate row

### Step 3: Check for Duplicates

Read the workbook (read_only=True) and check each target sheet for:
1. Exact name match
2. Similar name (fuzzy)
3. Same value + unit combination

Flag duplicates clearly in the validation UI.

### Step 4: Check Sources

Read `tblSources` and check each source:
1. If exists → note existing Source ID, still show for user confirmation
2. If new → prepare for creation, show for user validation

**Never auto-approve sources.** All sources appear in the validation UI with editable fields.

### Step 5: Present Validation UI

Use `show_widget` to render an interactive validation interface with inline-editable fields.

**Architecture of the widget:**

The widget must:
- Render all source cards first, then assumption cards
- Make every text field directly editable (contenteditable or input elements)
- Track card state (pending/ok/discard) with visual border colors
- Show a progress counter
- Include "Submit Results" AND "Additional Notes" at the bottom
- On submit, serialize ALL card data (including edits) as JSON and call `sendPrompt(json)`

**Source Cards** (appear first):
```
┌─────────────────────────────────────────────────┐
│ 📚 SOURCE #1                    [NEW / EXISTS]  │
│                                                 │
│ Title:  [editable text field                 ]  │
│ Author: [editable text field                 ]  │
│ Type:   [editable text field                 ]  │
│ Year:   [editable text field                 ]  │
│ Link:   [editable text field                 ]  │
│                                                 │
│  [✓ OK]              [✗ Discard]                │
└─────────────────────────────────────────────────┘
```

**Assumption Cards:**
```
┌─────────────────────────────────────────────────┐
│ #1                          [Target: dropdown]  │
│                                                 │
│ Assumption: [editable                        ]  │
│ Value:      [editable — numeric only         ]  │
│ Unit:       [editable                        ]  │
│ Source:     [editable — author name          ]  │
│ Observations: [editable                      ]  │
│ Internal:   [editable                        ]  │
│                                                 │
│ ⚠ DUPLICATE: H-012 "CAPEX Eletrolisador"       │
│   Existing value: 1400 USD/kW                   │
│                                                 │
│  [✓ OK]              [✗ Discard]                │
└─────────────────────────────────────────────────┘
```

**Border colors:**
- Gray: not yet validated
- Green: OK (approved)
- Red: Discard

**Bottom controls:**
- Progress bar: "X of Y validated"
- Text area: "Additional Notes" — general comments/instructions
- Button: "Submit Results" (enabled when all cards are validated)

**Submit behavior:**
The submit button MUST call `sendPrompt()` with a JSON string containing:
```json
{
  "action": "import_assumptions",
  "notes": "user's additional notes text",
  "sources": [
    {"status": "ok", "title": "...", "author": "...", ...},
    {"status": "discard", ...}
  ],
  "assumptions": [
    {"status": "ok", "sheet": "Hydrogen", "assumption": "...", "value": 1500, ...},
    {"status": "discard", ...}
  ]
}
```

This is CRITICAL. The JSON must contain the ACTUAL CURRENT VALUES of all editable fields, not the original values. The widget must read from the DOM at submit time.

### Step 6: Process Submitted Data

When receiving the JSON from the widget:

1. Parse the JSON — this contains the user's final validated data with all edits
2. Filter: keep only items with `"status": "ok"`
3. Read any additional notes and apply as global instructions
4. Present final summary:

```
Resumo da importação:
- X premissas para inserir (Y em Hydrogen, Z em CCUS, ...)
- N novas fontes para criar
- D itens descartados
Notas: "..."
Confirmar? [Sim] [Cancelar]
```

### Step 7: Write JSON Import File

**DO NOT WRITE TO THE .XLSM FILE.** Instead:

1. Write `_premissas_import.json` to the workbook folder:

```python
import json
data = {
    "sources": [...],  # approved sources
    "assumptions": [...]  # approved assumptions
}
with open('path/to/workbook/folder/_premissas_import.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
```

2. Tell the user:

> **Arquivo de importação criado: `_premissas_import.json`**
> 
> Para importar no workbook:
> 1. Abra o `Premissas_Decarb.xlsm` no Excel
> 2. Rode **Alt+F8 → ImportFromJSON**
> 3. O macro cria um backup automático antes de importar
> 4. Verifique os dados inseridos

### Step 8: Generate Report

Show a summary of what was exported to the JSON file.

## Edge Cases

### Temporal Projections
Base year value as main assumption. Full projection in Observations or as separate rows per year.

### Missing Information
Ask the user — never guess or leave blank.

### Multiple Sources for One Assumption
Most authoritative/recent as primary Source. List others in Observations.
