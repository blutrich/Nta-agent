---
name: show-map
description: Phase 3. Used for flows 1 and 2. Sends the city map image, asks for a site number, parses the response, and passes to lookup-contact. Triggers on current_flow = "active-site" or "future-site".
---

# show-map

Phase 3 of the NTA agent flow. Handles the map sub-flow for topics 1 and 2.

## When To Run

When `session.current_flow` is "active-site" or "future-site".
Also runs again on invalid site number (re-prompt).

## What It Does

### Step A — Send Map

1. Send the text prompt from menu-flows.md:
   - Flow 1: "זוהי מפת האתרים הפעילים בעירך. לאיזה מספר אתר תרצה לקבל מידע?"
   - Flow 2: "זוהי מפת האתרים המתוכננים בעירך. לאיזה מספר אתר תרצה לקבל מידע?"
2. Send the map image: `petah-tikva-map.png` (from Knowledge files)
3. Update `session.current_step` to "awaiting-site-number"

### Step B — Receive Site Number

When user replies with a number:

1. Parse integer from the message
2. Look up the number in `rules/site-directory.md`, matching the current flow:
   - Flow 1 → check "Active Sites" table
   - Flow 2 → check "Future Sites" table
3. If FOUND → extract: site_name, metro_line, excel_contact_row
4. If NOT FOUND → send error message, re-send map (run Step A again)

### Step C — Site Found Confirmation

Send the confirmation message from menu-flows.md, substituting real values:

```
✅ ביקשת מידע על אתר [N] — [site_name].
האתר הינו חלק מקו המטרו [metro_line].
לקבלת מידע על לוחות הזמנים [הנוכחיים/המתוכננים], ניתן לפנות לאגף [division_name] בנת"ע.
לחץ המשך לקבלת פרטי הקשר.
```

Add one inline button: **המשך →**

### Step D — Hand Off

When user taps "המשך":
→ Update session with `excel_contact_row` found in Step B
→ Run lookup-contact skill

## Error Handling

### Site number not in directory

```
לא מצאתי אתר במספר [N] במפה שלפניך.
אנא בדוק/י את מספר האתר ונסה/י שוב.
```

Re-send map image. Do not advance to lookup-contact.

### User sends text instead of number

→ Run handle-unknown skill

### Map image fails to send

Fall back to text list of site numbers from site-directory.md.

## Session State Updates

```yaml
session:
  current_step: "awaiting-site-number" | "awaiting-continue" | "done"
  selected_site_number: <int>
  selected_site_name: <string>
  selected_metro_line: <string>
  excel_contact_row: <int>
```
