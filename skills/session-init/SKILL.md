---
name: session-init
description: Phase 1. Runs on every /start command. Verifies the user is on the allowlist, initializes session state, and sends the greeting with the 7-option main menu. Triggers on "/start", "start", "התחל", "שלום".
---

# session-init

Phase 1 of the NTA agent flow. Runs first on every session.

## When To Run

On every `/start` command or when the user sends a fresh greeting to the bot.

## What It Does

1. Extract `telegram_user_id` and `telegram_first_name` from the incoming message
2. Check `telegram_user_id` against `rules/allowlist.md`
3. If NOT on the list → send rejection message → terminate
4. If on the list → initialize session state in Memory
5. Look up the user's pre-assigned city from the User section of Brain (for demo: Petah Tikva)
6. Send greeting message (from identity.md template)
7. Send 7-option inline keyboard (from menu-flows.md)
8. Deduplicate: check if this `message_id` was already processed. If yes, skip.

## Allowlist Check

Read `allowlist.md` from rules/. The check has two modes:

### OPEN_MODE: true (demo default)

Skip the ID check entirely. Every Telegram user proceeds straight to greeting + menu.

### OPEN_MODE: false (closed mode)

Compare incoming `message.from.id` (integer) against the PERMITTED_TELEGRAM_IDS list.

- Match → proceed
- No match → send this exact message and stop:

```
הבוט נמצא בשלב פיילוט סגור.
לפרטים ניתן לפנות להילה וקסברג, מנהלת אגף רשויות מקומיות.
📞 050-403-7303
```

When the operator wants to switch modes, they message the agent (e.g., "סגור את הבוט" → set OPEN_MODE: false; "פתח את הבוט" → set OPEN_MODE: true). The agent rewrites `allowlist.md` via write_file. No restart needed — Base44 reloads rules files every run.

## Session State (write to Memory)

```yaml
session:
  telegram_user_id: <int>
  telegram_first_name: <string>
  assigned_city: "Petah Tikva"   # demo: hardcoded. Phase 2: from auth flow
  current_flow: null              # null = at main menu
  current_step: null
  last_message_ids: []            # for deduplication
  started_at: <ISO timestamp>
```

## Greeting Message

Use the template from identity.md, substituting `telegram_first_name`:

```
שלום [first_name]! 👋
אני העוזר/ת הדיגיטלי/ת של אגף רשויות מקומיות בנת"ע.
[אני רואה שאת/ה מייצג/ת את עיריית פתח תקווה.]

איך אפשר לעזור לך היום? אפשר לכתוב לי בחופשיות מה את/ה צריך/ה,
או לבחור אחת מהאפשרויות:
```

Immediately followed by the 7-option keyboard (no gap).

The phrase "אפשר לכתוב לי בחופשיות" matters — it signals to the user that this is a conversation, not a form. Do not omit it.

## Keyboard Format

Telegram inline keyboard, one button per row, Hebrew text:

```
[ 1. אתר פעיל — עבודות נת"ע בביצוע      ]
[ 2. אתר עתידי — עבודות טרם החלו         ]
[ 3. תוכנית יזמית בגבולות תת"ל           ]
[ 4. סביבה ומיגון אקוסטי דירתי           ]
[ 5. נת"ע כרשות התמרור                   ]
[ 6. הסברה לציבור ומפגשי תושבים          ]
[ 7. נושא אחר — דבר עם מישהו בנת"ע      ]
```

## Output

- Session state written to Memory
- Greeting message sent to Telegram user
- 7-option keyboard sent to Telegram user
- Passes to route-user skill on next user message

## What NOT To Do

- Never show the menu to an unlisted user
- Never ask the user to verify their city during the demo (it's pre-assigned)
- Never skip the deduplication check
