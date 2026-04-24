# phase2-whatsapp.md

How to flip the NTA agent from Telegram to WhatsApp after CEO approval.
This is a Phase 2 deliverable — do not attempt before the demo is approved.

---

## What Needs To Change

Almost nothing. The Superagent Brain, skills, rules, and knowledge files are channel-agnostic.
The only change is the channel setting.

| What | Telegram (v0.1) | WhatsApp (Phase 2) |
|---|---|---|
| Channel setting | Settings → Channels → Telegram | Settings → Channels → WhatsApp |
| Setup time | 5 min (BotFather token) | 2 min (scan QR, send activation message) |
| User experience | Bot link / QR → Telegram app | QR → WhatsApp, send activation message |
| Inline keyboards | Native Telegram buttons | WhatsApp list messages or numbered options |
| Auth | Telegram user ID allowlist | Phone number → NTA rep list |

---

## Step-by-Step (Phase 2)

1. In the Superagent, go to Settings → Channels
2. Click "Continue on WhatsApp"
3. Click "Open WhatsApp"
4. Send the activation message shown to your WhatsApp number
5. Done — the same agent now responds on WhatsApp

---

## What Changes In The UX

WhatsApp doesn't support Telegram-style inline keyboards natively.
The 7-option menu becomes either:
- **Numbered text list** (user types "2" to select option 2) — simpler, matches Hila's original spec
- **WhatsApp list message** (native WhatsApp feature, shows a scrollable list) — better UX, slightly more config

For Phase 2, use the numbered text list approach since it requires no changes to the skill logic.

---

## Auth Upgrade (Phase 2)

Replace the Telegram-ID allowlist with phone-verified auth:

**Option A — Telegram phone share (if keeping Telegram):**
Send a Telegram keyboard button that requests the user's phone number.
Telegram delivers the phone number to the bot.
Check the phone against the NTA municipality rep list.

**Option B — SMS OTP (works for both Telegram and WhatsApp):**
User sends /start → bot asks for their phone number as text.
Bot sends a 6-digit OTP to that number via SMS (Twilio or similar).
User sends the OTP back → bot verifies → session opens.
Check the verified phone against the NTA rep list.
OTP codes expire in 5 minutes.

**Recommended for Phase 2:** Option A if staying on Telegram, Option B if moving to WhatsApp.

---

## Scaling To 24 Cities (Phase 2)

Current demo: Petah Tikva only.
site-directory.md is the only file that needs to grow.

To add a city:
1. Add a new section to site-directory.md with the city's site table
2. Add the city's map image to Knowledge files (e.g., ramat-gan-map.png)
3. session-init detects the user's city (from auth or from /start parameter)
4. show-map loads the correct city's map and site table

The contacts.xlsx may also need new rows for city-specific contacts,
but most contacts (M2/M3 line divisions) are the same across cities.

No code changes required for scaling. Only data.
