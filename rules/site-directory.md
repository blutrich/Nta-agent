# site-directory.md

Site number → metro line → phase → Excel contact row lookup table.
Auto-loaded into system prompt via .agents/rules/.

This file covers Petah Tikva sites for the v0.1 demo.
In Phase 2, add a section per city, loaded based on session city.

---

## Petah Tikva — Active Sites (Flow 1)

| Site # | Site Name | Metro Line | Phase | Excel Contact Row | Notes |
|--------|-----------|-----------|-------|-------------------|-------|
| 2 | סירקין צפון | M2 | ביצוע | Row 8 | M2 Execution Division |
| 3 | סירקין דרום | M2 | ביצוע | Row 8 | M2 Execution Division |
| 5 | קפלן | M2 | ביצוע | Row 8 | M2 Execution Division |

## Petah Tikva — Future Sites (Flow 2)

| Site # | Site Name | Metro Line | Phase | Excel Contact Row | Notes |
|--------|-----------|-----------|-------|-------------------|-------|
| 10 | בילינסון | M3 | תכנון | Row 12 | M3 Planning Division |
| 11 | קניון הגדול | M3 | תכנון | Row 12 | M3 Planning Division |
| 12 | מרכז העיר | M3 | תכנון | Row 12 | M3 Planning Division |

---

## Lookup Logic

1. User sends site number (e.g., "10")
2. Agent checks current flow context (Flow 1 = active, Flow 2 = future)
3. Agent looks up this table for a matching row
4. If found → use the Excel Contact Row to fetch contact from contacts.xlsx
5. If NOT found → send the "site not found" error message from menu-flows.md, re-send map

---

## PLACEHOLDER NOTE

This file uses placeholder data for the demo.
Hila must supply the actual site list from her Excel before Day 1 build starts.
Replace this entire table with real data when received.

The Excel row numbers (Row 8, Row 12) are also placeholders — they reference Hila's
contacts Excel row numbering. Update when the actual Excel file is received.
