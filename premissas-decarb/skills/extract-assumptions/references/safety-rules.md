# Safety Rules

## GOLDEN RULE

**NEVER open, modify, or save the workbook file (.xlsm) directly via Python, openpyxl, or any file-writing library.**

openpyxl DESTROYS shapes, buttons, and visual elements when saving .xlsm files. This is a known, unfixable limitation. The ONLY safe way to write data into this workbook is through Excel itself, using the workbook's own VBA import system.

The plugin writes a `.json` import file. The user then runs a VBA macro inside Excel that reads the JSON and uses the workbook's own functions to insert data properly.

## Absolute Prohibitions

1. **Never open the .xlsm file for writing** — not with openpyxl, xlsxwriter, or any library. READING with `read_only=True` is OK.
2. **Never modify VBA code, shapes, buttons, or visual elements**
3. **Never write to ID columns** — column 1 of any table
4. **Never write to the Change Log**
5. **Never delete, move, rename, or create sheets**
6. **Never change protection, named ranges, data validation, or conditional formatting**
7. **Never modify the _Lang translation table**
8. **Never modify formulas or existing data**

## Permitted Operations

1. **Read the workbook** (openpyxl with `read_only=True`) to check existing data, find duplicates, count rows
2. **Write a `_premissas_import.json`** file to the workbook folder
3. **Instruct the user** to run the VBA import macro in Excel

## Write Procedure

1. Validate all data with the user via the interactive widget
2. Write `_premissas_import.json` to the workbook folder:
```json
{
  "sources": [
    {"title": "...", "author": "...", "type": "Report", "pub_date": "2024-01-15", "link": "https://..."}
  ],
  "assumptions": [
    {
      "sheet": "Hydrogen",
      "assumption": "CAPEX Eletrolisador PEM",
      "value": 1500,
      "unit": "USD/kW",
      "source_author": "IEA",
      "access_date": "2025-06-02",
      "internal": "No",
      "responsible": "Lucas Bacellar",
      "observations": "Base case, utility-scale"
    }
  ]
}
```
3. Tell the user: **"Abra o workbook no Excel e rode Alt+F8 → ImportFromJSON"**
4. The VBA macro handles: backup, source creation, assumption insertion, ID generation, table expansion, protection

## Data Integrity Rules

### One Assumption = One Value = One Row
ALWAYS. Never combine multiple data points into one assumption.
- "Trucks: Volvo 300 km, Scania 250 km" → TWO separate rows
- "CAPEX: 1500 USD/kW (2025), 1200 USD/kW (2030)" → TWO separate rows (or one with base year + projection in observations)

### Value Field = Numeric Only
- ONLY numbers. No text, no ranges, no units.
- Decimal separator is ALWAYS "." (dot) — Excel is in English
- Valid: `1500`, `0.85`, `275.5`
- Invalid: `"1,500"`, `"250-300"`, `"1500 USD/kW"`

### Range Values → Average
Range "250 - 300" → Value: `275`, Observations: "Average of range 250-300 from source"

### Observations That Contain Data → Separate Assumption
If an observation has a quantitative value (e.g., "Consumo médio de 0.8 a 1.1 km/m³"), extract as its own assumption row:
- Value: `0.95` (average)
- Observations: "Average of range 0.8-1.1"

### Source Handling
- Plugin NEVER generates Source IDs — VBA does this
- In JSON, reference sources by `source_author`
- VBA import macro matches or creates the source
