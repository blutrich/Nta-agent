# contacts-schema.md

How Hila's Excel file maps to the agent's contact lookups.
This file helps Ofer ensure the Excel columns are read correctly by lookup-contact/SKILL.md.

---

## Actual Excel Column Schema (from Hila's file, April 24 2026)

The contacts file has no column headers — columns are positional:

| Column | Content | Maps To |
|--------|---------|---------|
| A (Unnamed:0) | שם (full name) | contact_name |
| B (Unnamed:1) | אגף / קו | division_name |
| C (Unnamed:2) | תפקיד (role) | role |
| D (Unnamed:3) | אימייל | email |
| E (Unnamed:4) | טלפון | phone |

The agent should read by column position (0-indexed: 0,1,2,3,4), not by header name.
working_hours is not in the file — omit that field from the contact card, or use "א-ה, 08:00-17:00" as default.

## Real Contact Rows (from Hila's Excel)

| row_id | Name | Division | Role | Email | Phone |
|--------|------|----------|------|-------|-------|
| 1 | הילה וקסברג | רשויות מקומיות | מנהלת אגף רשויות מקומיות | HilaW@nta.co.il | 050-403-7303 |
| 2 | הדר אבניאל | רשויות מקומיות | מנהלת רשויות מקומיות | hadara@nta.co.il | 053-530-5608 |
| 3 | זוהרה ישי | תיאום תשתיות ותוכניות גובלות | מנהלת אגף תיאום תשתיות ותוכניות גובלות | zoharai@nta.co.il | 053-530-5604 |
| 4 | עדי קין קרני | M2 | מנהלת קו M2 | adika@nta.co.il | 053-530-5605 |
| 5 | מיכל זוהר | M2 | מנהלת אגף מקרקעין M2 | michalzoh@nta.co.il | 053-530-5606 |
| 6 | דקלה אסרף | M2 | קשרי קהילה M2 | xxx@abc.co.il | 053-530-5607 |
| 7 | דור נדל | M2 | מנהל מערך תכנון M2 | dorn@nta.co.il | 053-530-5609 |
| 8 | טל מלכה | M2 | מנהל אגף הקמה M2 | talma@nta.co.il | 053-530-5610 |
| 9 | לאה שמול | M3 | מנהלת קו M3 | leas@nta.co.il | 053-530-5611 |
| 10 | וארחי ברדיין | M3 | מנהל אגף מקרקעין M3 | orchaib@nta.co.il | 053-530-5612 |
| 11 | איציק [קשרי קהילה] | M3 | קשרי קהילה M3 | xxx@abc.co.il | 053-530-5613 |
| 12 | רחלי ברגר | M3 | מנהלת אגף תכנון M3 | rachelb@nta.co.il | 053-530-5614 |
| 13 | שני שקד | M3 | מנהלת אגף הקמה M3 | shanis@nta.co.il | 053-530-5615 |

Note: Rows 6 and 11 (קשרי קהילה) have placeholder emails (xxx@abc.co.il). Hila to confirm real emails before demo.

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
