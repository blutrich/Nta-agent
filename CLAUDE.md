# CLAUDE.md

Project memory for any AI agent (Claude, Base44 Superagent, etc.) working with this repo.

## What This Repo Is

A Base44 Superagent configuration bundle that turns a new Superagent into the **NTA Local Authorities Chat Agent** — a deterministic routing assistant for representatives from Israel's 24 partner municipalities. The agent runs on Telegram for the v0.1 demo, and can be flipped to WhatsApp in Phase 2 via a single channel setting.

**This is not a conversational AI agent. It is a state machine with a chat skin.** The Superagent's LLM is intentionally constrained to follow a closed decision tree. Every response comes from the contacts Excel file or a pre-written menu-flows file. The LLM never improvises.

## Core Concept: The 5-Phase Flow

Every user interaction runs this sequence:

```
PHASE 1: session-init     → verify user is in allowlist, greet by name, confirm city
PHASE 2: route-user       → present 7-option menu, parse user selection
PHASE 3: show-map         → for topics 1+2, send city map image, ask for site number
PHASE 4: lookup-contact   → resolve topic+site → Excel row → contact card
PHASE 5: handle-unknown   → any input not matching a menu option → re-prompt or fallback
```

## Repo Structure

```
nta-agent/
├── CLAUDE.md                        # This file
├── BOOTSTRAP-PROMPT.md              # Single message to paste into a new Superagent
├── RECOVERY-PROMPTS.md              # Fix broken installs
├── soul.md                          # Behavioral principles — deterministic, never improvise
├── identity.md                      # NTA bot identity (Hebrew, name, avatar, tone)
├── verify-install.md                # Test checklist
├── rules/
│   ├── menu-flows.md                # Authoritative Hebrew wording for all 7 flows
│   ├── site-directory.md            # Site number → metro line → Excel row lookup
│   ├── allowlist.md                 # Demo: 3 permitted Telegram user IDs
│   └── guardrails.md                # Hard rules: never invent contacts, always Excel
├── skills/
│   ├── session-init/SKILL.md        # Phase 1: verify allowlist, greet, set session
│   ├── route-user/SKILL.md          # Phase 2: show menu, parse selection
│   ├── show-map/SKILL.md            # Phase 3: send map image, parse site number
│   ├── lookup-contact/SKILL.md      # Phase 4: Excel row → contact card
│   └── handle-unknown/SKILL.md      # Phase 5: handle free text, bad input, fallback
├── tasks/
│   └── session-timeout.md           # Clear session after 30 min idle
└── docs/
    ├── contacts-schema.md            # How Hila's Excel maps to the agent's lookups
    └── phase2-whatsapp.md            # How to flip channel to WhatsApp post-demo
```

## Critical Design Rules

### 1. Deterministic over Intelligent

The agent NEVER invents answers. It is a router, not an advisor. If the Excel row is missing, it falls back to Hila's contact (row 2). If the user's input doesn't match anything, it re-prompts with the menu.

**Never:**
- Generate contact names, phone numbers, or emails from LLM knowledge
- Speculate about NTA project timelines or policy
- Answer questions outside the 7 menu topics
- Add helpful commentary to contact cards ("this division is very responsive")

**Always:**
- Pull contact info verbatim from the Excel knowledge file
- Return to the main menu after every completed interaction
- Address the user by their Telegram first name throughout the session

### 2. Allowlist Is The Gate (Demo Grade)

For v0.1: the agent checks `telegram_user_id` against `rules/allowlist.md` on every `/start`. Users not on the list receive:

> הבוט נמצא בשלב פיילוט סגור. לפרטים ניתן לפנות להילה וקסברג, מנהלת אגף רשויות מקומיות.

The allowlist is intentionally simple. Phase 2 replaces it with phone-verified OTP against an NTA-maintained rep list.

### 3. Hebrew First, RTL Always

All user-facing text is Hebrew. The agent never switches to English mid-conversation. Buttons, menus, contact cards, error messages — all Hebrew. The only exception: the technical SKILL.md files in this repo are written in English for developer readability.

### 4. Silence Is Better Than Wrong Info

If any lookup returns no result (missing Excel row, unrecognized site number, empty contact field), the agent does not guess. It returns the fallback contact (Hila, row 2) and says:

> לא מצאתי מידע ספציפי עבור הפנייה הזו. ניתן לפנות ישירות למנהלת אגף רשויות מקומיות.

### 5. No Persistence Between Demos (Demo-Only Rule)

For the 27.4 demo: session state lives in Superagent short-term memory only. After `/end` or 30 min idle, session clears. This is intentional — each demo attendee gets a clean slate.

### 6. File Writes Must Use write_file (Base44 Critical Gotcha)

Base44 Superagents do NOT persist files written via bash `cat >`, `cp`, or `echo >`. Those writes go to an ephemeral sandbox and vanish on restart.

**Only `write_file` persists to Base44 Files storage.**

Skills MUST be folders: `.agents/skills/{name}/SKILL.md`
Rules MUST be at: `.agents/rules/{name}.md`
Never create scripts, never create .sh/.py/.js files. The markdown IS the skill.

## How The Demo Install Works

1. Operator (Ofer) creates a new Base44 Superagent
2. Pastes `BOOTSTRAP-PROMPT.md` verbatim into the chat
3. Answers the 4 setup questions (bot name, Telegram token, Hila's Excel file, 3 test user IDs)
4. Watches 10 ✅ markers run
5. Runs `verify-install.md` test checklist
6. Shares the Telegram bot link with Hila for pre-demo review (April 26)
7. Demo runs April 27

## Critical External Dependencies

| Dependency | Purpose | Demo Required | Fallback |
|---|---|---|---|
| Base44 Superagent | Runtime | Yes | None |
| Telegram Bot (BotFather) | Channel | Yes | None |
| contacts.xlsx | Source of truth for all contact cards | Yes | None |
| petah-tikva-map.png | Site map for topics 1+2 | Yes | Text list of sites |
| WhatsApp (Base44 native) | Phase 2 channel | No | — |

## What To Watch Out For

1. **Superagents are agentic by default.** The soul.md + guardrails.md are what make the agent stay deterministic. Test aggressively: ask it free-text questions and confirm it re-prompts rather than answering.
2. **write_file only.** Never bash for file creation under .agents/.
3. **Hebrew keyboard display.** Telegram inline keyboard buttons render RTL correctly. Verify on Day 1.
4. **Duplicate Telegram webhooks.** Telegram occasionally fires the same message twice. The session-init skill deduplicates by `message_id`.
5. **Missing Excel rows.** If Hila's Excel has empty rows for topics 4-6, the fallback to row 2 (Hila) must trigger cleanly.
