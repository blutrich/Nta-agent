# allowlist.md

Demo-grade access control for v0.1.
Auto-loaded into system prompt via .agents/rules/.

---

## Permitted Telegram User IDs

This list contains the Telegram numeric user IDs allowed to use the bot during the demo.
Any Telegram user not on this list receives the rejection message and cannot proceed.

The list **ships empty on purpose**. The operator (Ofer) adds IDs after the bot is live by sending the agent a Hebrew command like `תוסיף את 123456789`. The agent updates this file via `write_file` and reloads.

```
PERMITTED_TELEGRAM_IDS:
  # empty — add IDs via "תוסיף את <id>" after install
```

When the operator sends an ID, the agent appends it under the comment. Example after one user is added:

```
PERMITTED_TELEGRAM_IDS:
  - 123456789   # Ofer (added 2026-04-25)
```

---

## How To Find A Telegram User ID

Option 1: Ask the user to message @userinfobot on Telegram. The bot replies with their numeric ID.
Option 2: Use the Telegram Bot API: the `message.from.id` field in any incoming message is the user's numeric ID.
Option 3: Ask Hila to share her Telegram ID by forwarding any message to @userinfobot.

---

## How The Allowlist Check Works

The session-init skill reads this file on every `/start`.
It compares the incoming `message.from.id` (Telegram numeric ID) against the list.

- Match found → proceed to greeting and menu
- No match → send rejection message, terminate session

---

## Rejection Message

> הבוט נמצא בשלב פיילוט סגור.  
> לפרטים ניתן לפנות להילה וקסברג, מנהלת אגף רשויות מקומיות.  
> 📞 050-403-7303

---

## Phase 2 Note

This file is replaced in Phase 2 with a proper authentication flow:
- User shares phone number via Telegram's native phone-share button
- Bot sends a 6-digit OTP to that phone via SMS
- Bot checks the verified phone against the NTA-maintained municipality rep list
- No hardcoded IDs — the list is pulled from the NTA system dynamically

The allowlist.md file is a demo shortcut, not a production mechanism.
