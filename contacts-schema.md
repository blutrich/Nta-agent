# contacts-schema.md

How Hila's Excel file maps to the agent's contact lookups.
This file helps Ofer ensure the Excel columns are read correctly by lookup-contact/SKILL.md.

---

## Expected Excel Column Schema

The agent reads contacts.xlsx and expects these columns (in any order — matched by header name):

| Column Header | Used For | Required |
|---|---|---|
| row_id | Internal reference (integer) | Yes |
| division_name | Contact card header | Yes |
| contact_name | 👤 field in contact card | Yes |
| role | 🏢 field in contact card | Yes |
| phone | 📞 field in contact card | Yes |
| email | 📧 field in contact card | Yes |
| working_hours | 🕐 field in contact card | Recommended |
| notes | Internal only, not shown to user | Optional |

---

## Critical Row Mappings

These rows are hardcoded in skill references and site-directory.md:

| Row ID | Who | Used For |
|--------|-----|---------|
| 2 | הילה וקסברג — מנהלת אגף רשויות מקומיות | Fallback for all flows. Topic 7 forced. |
| 3 | Infrastructure Coordination Division | Topic 3 (development plans bordering metro) |
| 8 | M2 Execution Division | Active sites on M2 line (flow 1, site-directory lookup) |
| 12 | M3 Planning Division | Future sites on M3 line (flow 2, site-directory lookup) |

**These row IDs are PLACEHOLDERS.** They were inferred from Hila's April 17 email.
Ofer must verify the actual row numbers against the real Excel file before Day 1 build.

---

## Topics Without Confirmed Excel Rows (needs Hila)

| Topic | Division Needed |
|-------|----------------|
| 4 | Environment / Acoustic Shielding Division |
| 5 | Traffic Signage Authority (רשות התמרור) |
| 6 | Public Communications / Resident Outreach |

If these rows don't exist in the Excel by Day 2, topics 4-6 will display the Hila fallback
with the prefix "לא מצאתי פרטי קשר ספציפיים". The demo still works — it just shows topic 7
behavior for these three.

---

## How lookup-contact Reads The File

1. Skill receives `excel_contact_row` (integer) from session state
2. Opens contacts.xlsx from Knowledge files
3. Finds the row where `row_id == excel_contact_row`
4. Reads the 6 fields listed above
5. If row not found OR any required field empty → uses row 2 (Hila) as fallback
6. Formats and sends the contact card template

---

## If Hila's Excel Uses Different Column Names

Update lookup-contact/SKILL.md's column references to match the actual headers.
Do not rename Hila's columns — adapt the skill to her file.
