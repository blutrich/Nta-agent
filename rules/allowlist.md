# allowlist.md

Demo-grade access control for v0.1.
Auto-loaded into system prompt via .agents/rules/.

---

## Permitted Telegram User IDs

This list contains the Telegram numeric user IDs allowed to use the bot during the demo.
Any Telegram user not on this list receives the rejection message and cannot proceed.

```
PERMITTED_TELEGRAM_IDS:
  - PLACEHOLDER_USER_ID_1   # Hila Wechsberg (test user 1)
  - PLACEHOLDER_USER_ID_2   # Test user 2 — fill in before Day 1 build
  - PLACEHOLDER_USER_ID_3   # Test user 3 — fill in before Day 1 build
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
