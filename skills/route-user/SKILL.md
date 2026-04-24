---
name: route-user
description: Phase 2. Receives the user's menu selection (1-7) and routes to the correct next skill. Triggers on menu button taps, number inputs 1-7, "חזרה לתפריט".
---

# route-user

Phase 2 of the NTA agent flow. Routes the user's selection to the correct next phase.

## When To Run

When the user taps one of the 7 menu buttons, or sends a number 1-7 while in the main menu state.

## What It Does

1. Read `session.current_flow` from Memory — confirm we are at the main menu (null)
2. Parse the incoming message or callback_data (button tap) for a number 1-7
3. Update `session.current_flow` in Memory to the selected topic
4. Route to the correct next skill

## Routing Table

| Input | Sets current_flow | Next skill |
|-------|------------------|-----------|
| 1 | "active-site" | show-map |
| 2 | "future-site" | show-map |
| 3 | "development-plan" | lookup-contact (row 3) |
| 4 | "environment" | lookup-contact (env row) |
| 5 | "traffic-signage" | lookup-contact (traffic row) |
| 6 | "public-outreach" | lookup-contact (outreach row) |
| 7 | "other" | lookup-contact (row 2 = Hila, forced) |

## Free Text Handling

If the incoming message is not a number 1-7 and is not a recognized command:
→ do NOT route anywhere
→ run handle-unknown skill instead

## Session Update

Write to Memory:
```yaml
session:
  current_flow: "<selected topic string>"
  current_step: "map" (for flows 1+2) or "contact" (for flows 3-7)
```

## What NOT To Do

- Never attempt to interpret free-text intent ("I want to know about a site" → NOT parsed as topic 1)
- Never show a partial menu or a subset of options
- Never skip updating session state before routing
