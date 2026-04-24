# allowlist.md

Demo-grade access control for v0.1.
Auto-loaded into system prompt via .agents/rules/.

---

## Mode

```
OPEN_MODE: true
```

When `OPEN_MODE: true`, **every Telegram user is allowed**. The session-init skill skips the ID check entirely. This is the demo default — share the bot link freely with Hila, the CEO, anyone.

When `OPEN_MODE: false`, only IDs in PERMITTED_TELEGRAM_IDS get through. Everyone else receives the rejection message.

To close the bot back down, edit this file (via write_file) and set `OPEN_MODE: false`.

---

## Permitted Telegram User IDs (used only when OPEN_MODE is false)

```
PERMITTED_TELEGRAM_IDS:
  # add IDs here when switching to closed mode
  # format:
  # - 123456789   # Name (added YYYY-MM-DD)
```

When the operator sends `תוסיף את <id>`, the agent appends the ID here. Adding IDs does not automatically close the bot — `OPEN_MODE` must be set to `false` separately.

---

## How To Find A Telegram User ID (when running in closed mode)

- Message `@userinfobot` on Telegram → it replies with the numeric ID
- Or, use the Telegram Bot API: the `message.from.id` field in any incoming message is the user's numeric ID

---

## How session-init Reads This File

On every `/start`:

1. If `OPEN_MODE: true` → proceed to greeting and menu (skip ID check)
2. Else, compare incoming `message.from.id` against `PERMITTED_TELEGRAM_IDS`
   - Match → proceed
   - No match → send rejection message, terminate session

---

## Rejection Message (closed mode only)

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

The OPEN_MODE shortcut is for demo only. Do not ship to Phase 2 with OPEN_MODE: true.
