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
3. Extract: division_name, contact_name, role, phone, email, working_hours
4. Check that none of these fields are empty
5. If any field is empty → use fallback (row 2, Hila)
6. Format and send the contact card
7. Send the "חזרה לתפריט הראשי" button
8. Clear `session.current_flow` and `session.current_step` in Memory (ready for next topic)

## Row Mapping (from route-user)

| Flow | Forced Row | Notes |
|------|-----------|-------|
| 1 (active site) | from site-directory.md lookup | varies by site |
| 2 (future site) | from site-directory.md lookup | varies by site |
| 3 (development plan) | Row 3 | Infrastructure Coordination |
| 4 (environment) | env row (TBD from Hila's Excel) | fallback to row 2 if empty |
| 5 (traffic signage) | traffic row (TBD) | fallback to row 2 if empty |
| 6 (public outreach) | outreach row (TBD) | fallback to row 2 if empty |
| 7 (other) | Row 2 | Always Hila — forced |

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

## Fallback Contact (Row 2 — Hila)

When any field is missing or an empty row is returned, send this instead:

```
לא מצאתי פרטי קשר ספציפיים לפנייה זו.
ניתן לפנות ישירות למנהלת אגף רשויות מקומיות:

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
