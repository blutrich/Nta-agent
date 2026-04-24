---
name: lookup-contact
description: Phase 4. Reads the contacts.xlsx Knowledge file and returns a formatted contact card. Called for all 7 flows after routing or site selection. Triggers on "lookup contact", "המשך", "phase 4", or directly from route-user and show-map.
---

# lookup-contact

Phase 4 of the NTA agent flow. Fetches the contact from the Excel file and delivers the contact card.

## When To Run

- After route-user (for flows 3, 4, 5, 6, 7)
- After show-map's "המשך" button is tapped (for flows 1 and 2)

## What It Does

1. Read `session.excel_contact_row` from Memory to determine which row to fetch
2. Open `contacts.xlsx` from Knowledge files
3. **Read by column position** — the file has no headers. Columns are: A (0)=name, B (1)=division/line, C (2)=role, D (3)=email, E (4)=phone. See `docs/contacts-schema.md`.
4. **Row indexing is 0-based.** `session.excel_contact_row = 0` means the first data row in the file. The route-user table uses the same convention.
5. Build the card fields: `contact_name=A`, `division_name=B`, `role=C`, `email=D`, `phone=E`. **There is no `working_hours` column** — use the default `א-ה, 08:00-17:00` for every card.
6. Check that name, role, phone, email are non-empty
7. If any of those fields are empty OR the row index is out of range → use fallback (row index 0, the first data row in whatever Excel was actually uploaded)
8. Format and send the contact card
9. Send the "חזרה לתפריט הראשי" button
10. Clear `session.current_flow` and `session.current_step` in Memory (ready for next topic)

## Row Mapping (from route-user)

| Flow | Row | Notes |
|------|-----|-------|
| 1 (active site) | from site-directory.md lookup | varies by site (M2 active sites → row 8 Tal Malka) |
| 2 (future site) | from site-directory.md lookup | varies by site (M2 future → row 7 Dor Nadel; M3 future → row 12 Racheli Berger) |
| 3 (development plan) | 3 | Zohara Yishai — Infrastructure Coordination |
| 4 (environment) | 3 (temp) | Routes to Zohara until dedicated env contact added |
| 5 (traffic signage) | 3 (temp) | Routes to Zohara until dedicated signage contact added |
| 6 (public outreach) | 6 (M2) or 11 (M3) | Dikla Asraf or Itzik per metro line |
| 7 (other) | 1 | Always Hila — forced |

## Contact Card Format

Use EXACTLY this template. No additions, no decorations, no paraphrasing.

```
📋 פרטי קשר — [division_name]

👤 [contact_name]
🏢 [role]
📞 [phone]
📧 [email]
🕐 שעות פעילות: [working_hours]
```

Followed immediately by one inline button: **חזרה לתפריט הראשי ↩**

## Fallback Contact (Row Index 0 — first data row in the actual Excel)

When any field is missing or the requested row is out of range, fall back to the **first data row of contacts.xlsx** (row index 0). Whoever sits at that row becomes the fallback. The current Excel from Hila has Hadar Avniel at row 0; if Hila is moved to row 0 later, the fallback flips to Hila automatically.

The card uses the same template as a normal lookup, prefixed with the apology line:

```
לא מצאתי פרטי קשר ספציפיים לפנייה זו.
ניתן לפנות ישירות לאגף רשויות מקומיות:

📋 פרטי קשר — [division from row 0]

👤 [name from row 0]
🏢 [role from row 0]
📞 [phone from row 0]
📧 [email from row 0]
🕐 שעות פעילות: א-ה, 08:00-17:00
```

If contacts.xlsx is unreachable entirely, send this hardcoded last-resort card (Hila's known details from Hila's email of April 17, 2026):

```
שירות התמיכה זמני אינו זמין. ניתן לפנות ישירות:

📋 פרטי קשר — אגף רשויות מקומיות

👤 הילה וקסברג
🏢 מנהלת אגף רשויות מקומיות
📞 050-403-7303
📧 HilaW@nta.co.il
🕐 שעות פעילות: א-ה, 08:00-17:00
```

[button: חזרה לתפריט הראשי ↩]

## CRITICAL Rules

- Phone numbers are copied VERBATIM from the Excel. No reformatting.
- Email addresses are copied VERBATIM. No autocorrect.
- division_name, contact_name, and role are copied VERBATIM. No translation.
- If contacts.xlsx is unavailable, use the Hila fallback hardcoded above.
- Never type contact details from LLM training data.

## Session Reset After Delivery

```yaml
session:
  current_flow: null
  current_step: null
  selected_site_number: null
  selected_site_name: null
  excel_contact_row: null
```

The session is now ready for the next topic selection.
