# CLAUDE.md

Project memory for any AI agent (Claude, Base44 Superagent, etc.) working with this repo.

## What This Repo Is

A Base44 Superagent configuration bundle that turns a new Superagent into the **NTA Local Authorities Chat Agent** — a Hebrew-speaking digital assistant for representatives from Israel's 24 partner municipalities. The agent runs on Telegram for the v0.1 demo, and can be flipped to WhatsApp in Phase 2 via a single channel setting.

**The agent is a conversational assistant with hard grounding rules.** Users can ask in free-text Hebrew ("מי אחראי על מיגון אקוסטי?") or tap one of the 7 menu buttons. Either way, the agent classifies intent, optionally answers briefly from documented procedures (`docs/nta-procedures.md`), and returns a contact card pulled verbatim from `contacts.xlsx`. The LLM can phrase its responses naturally — it **cannot** invent contact details, speculate on timelines, or answer questions outside NTA routing scope.

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
└── docs/
    ├── contacts-schema.md            # How Hila's Excel maps to the agent's lookups
    ├── nta-procedures.md             # PE-100/PE-060 procedural facts (topics 3, 4)
    └── phase2-whatsapp.md            # How to flip channel to WhatsApp post-demo
```

**Note on session timeout:** RULE G-5 in `rules/guardrails.md` mandates a 30-min idle timeout. For the demo, this is implemented via Base44 Memory's natural session expiration rather than as a separate scheduled task. If a hard timeout is needed in Phase 2, add it as `tasks/session-timeout.md`.

## Critical Design Rules

### 1. Conversational Tone, Hard Grounding

The agent speaks naturally in Hebrew — it can acknowledge, bridge, and respond to free-text requests. What it CANNOT do is invent grounding data.

**Never:**
- Generate contact names, phone numbers, or emails from LLM knowledge — always read from `contacts.xlsx`
- Speculate about NTA project timelines, approvals, or apartment-specific eligibility
- Answer procedural questions beyond what's documented in `docs/nta-procedures.md` (PE-100, PE-060)
- Add unfounded commentary to contact cards ("this division is very responsive")
- Mechanically re-send the 7-option menu on every free-text message — that feels like a broken phone tree

**Always:**
- Pull contact info verbatim from the Excel knowledge file
- Offer an open next step after a contact card delivery
- Address the user by their Telegram first name on greeting (not every turn)
- Route out-of-scope questions gracefully to Hila — don't answer them from LLM knowledge

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

### 7. Disk Presence ≠ Brain Registration (the #1 install failure)

Base44 Superagents have TWO separate places skills/rules live:

1. `.agents/skills/` and `.agents/rules/` folders on disk — easy to write
2. **Brain → Integrations → Skills** and **Brain → Rules → Rules files** — what the runtime actually uses

When told "create a skill", agents often write to disk without registering in Brain. The skill exists but the runtime can't see it. Symptom: bot responds but ignores all skills/rules — acts like a generic Base44 bot.

**Always verify both:** `find /app/.agents/skills -type f` should show 5 SKILL.md files, AND Brain → Integrations → Skills tab should show count = 5. Same for rules: 4 files on disk AND 4 in Brain → Rules.

If Brain UI count is 0 but disk count is right, see RECOVERY-PROMPTS.md Recovery 8.

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
3. **Disk ≠ Brain.** Files in `.agents/skills/` and `.agents/rules/` must also be registered in Brain → Integrations → Skills and Brain → Rules. See Critical Design Rule #7.
4. **Hebrew keyboard display.** Telegram inline keyboard buttons render RTL correctly. Verify on Day 1.
5. **Duplicate Telegram webhooks.** Telegram occasionally fires the same message twice. The session-init skill deduplicates by `message_id`.
6. **Missing Excel rows fall back to Hila (row 1).** If Hila's Excel has empty rows for topics 4-6, the fallback must trigger cleanly. Note: Hila is row 1, NOT row 2 — older drafts of skill files may say row 2.
7. **Rows 7 and 8 are easy to swap.** Row 7 = Dor Nadel (M2 Planning, future sites). Row 8 = Tal Malka (M2 Execution, active sites). Active vs. future is the discriminator.

## Conventions for Editing This Repo

### When adding a new skill

1. Create `skills/{name}/SKILL.md` with YAML frontmatter (name + description with trigger phrases)
2. Add it to `BOOTSTRAP-PROMPT.md` Step 5 skill list (and update the "expect exactly N SKILL.md files" count)
3. Add it to `verify-install.md` Test 1 expected list
4. Add it to `README.md` repo layout if user-visible
5. Update CLAUDE.md repo structure section
6. Update `soul.md` Decision-Making section if the new skill changes the routing logic
7. Test the install with a fresh Superagent before committing

### When adding a new rules file

1. Create `rules/{name}.md`
2. Add to `BOOTSTRAP-PROMPT.md` Step 4 list
3. Add to `verify-install.md` Test 2 expected list
4. Add to `README.md` repo layout
5. Update CLAUDE.md repo structure
6. Remember: every file under `.agents/rules/` is auto-loaded into the system prompt every run, so keep it small and high-signal

### When the Excel schema changes

If Hila revises contacts.xlsx:

1. Update `docs/contacts-schema.md` "Actual Excel Column Schema" table
2. Update `docs/contacts-schema.md` "Critical Row Mappings" table if rows shifted
3. Update `rules/site-directory.md` row_id references for active + future sites
4. Update `rules/site-directory.md` "Contact Row Reference" table at the bottom
5. Update `skills/lookup-contact/SKILL.md` "Row Mapping" table
6. Update `skills/route-user/SKILL.md` Routing Table if topic→row mappings changed
7. Re-run `verify-install.md` Test 4 (happy path) end-to-end with a real allowlisted Telegram user

### When changing the menu (adding/removing/renaming a topic)

1. Update `rules/menu-flows.md` — main menu buttons + flow definitions
2. Update `skills/route-user/SKILL.md` Routing Table
3. Update `skills/session-init/SKILL.md` keyboard format
4. Update `BOOTSTRAP-PROMPT.md` Step 10 dry-run scenarios
5. Update `verify-install.md` test scenarios that reference specific topics
6. Update `README.md` "The 7 topics" list

### When updating soul.md

`soul.md` defines deterministic-router behavior. Changes affect every interaction.

**Safe changes:**
- Adding new banned chatty phrases to the Personality Override section
- Adding examples to existing principles
- Clarifying ambiguous decision-making rules

**Dangerous changes:**
- Removing the CRITICAL: Personality Override section (default Base44 chatty bot will leak through)
- Allowing free-text answers
- Removing the "fallback always to Hila row 1" rule
- Allowing the agent to skip the menu

If you change soul.md, test with a fresh Superagent install to verify the personality override still kills the default chatty bot.

## Common Operations

### "I want to add a new municipality"

1. Add a new section to `rules/site-directory.md` with that city's site numbers and row mappings
2. Upload a city map PNG to Knowledge files
3. Update `skills/show-map/SKILL.md` to look up the map by `session.assigned_city`
4. Update the bootstrap allowlist with that city's reps
5. (Phase 2 only) Update the auth flow to associate phone-verified IDs with cities

### "I want to update the contacts file"

Hila replaces contacts.xlsx in the Knowledge files. The bot picks it up on the next interaction. No skill/rule changes needed unless the column layout or row order changed (see "When the Excel schema changes" above).

### "I want to switch from Telegram to WhatsApp"

See `docs/phase2-whatsapp.md`. In short: change the channel setting in Base44 from Telegram to WhatsApp. The skills are channel-agnostic (they read message text + send buttons). Only `session-init`'s allowlist check needs to change from Telegram user ID to phone number.

### "I want to pause the bot temporarily"

In Base44 → Settings → Channels → Telegram → Disable. The bot stops responding. Re-enable when ready. No data lost.

### "I want to debug a wrong contact card"

1. Identify which row was returned (check Memory or session log)
2. Open contacts.xlsx and read that row by position (A=name, B=division, C=role, D=email, E=phone)
3. Compare what the bot sent vs. what the file actually contains
4. If they differ → guardrail break, see RECOVERY-PROMPTS.md Recovery 5
5. If they match but feel wrong → wrong row was selected. Check `route-user`'s Routing Table or `site-directory.md`'s mapping for that site number.

### "Hila's Telegram ID changed / a new test user needs access"

Update `rules/allowlist.md` PERMITTED_TELEGRAM_IDS list, then re-write to `.agents/rules/allowlist.md` via write_file (NOT bash). Brain auto-reloads on next /start.

## Recent Architectural Decisions

These are decisions Ofer has made in conversation. Future agents working on this repo should preserve them unless explicitly told to change them.

- **2026-04-25: Shift from deterministic router to conversational assistant.** After the first install, Ofer's feedback was "the experience is not fun, it feels like a CS bot". Soul.md, identity.md, and the skills were rewritten to handle free-text Hebrew requests naturally. The 7-option menu is still presented on greeting as a starting point, but it's no longer the only input mode. Hard grounding rules (never invent contacts, only cite documented procedures) are preserved.
- **2026-04-25: Allowlist defaults to OPEN_MODE: true for demo.** The bot lets anyone in by default. Closed mode is preserved as a toggle for post-demo lockdown.
- **2026-04-24: Excel schema is positional, not header-based.** The real contacts.xlsx has no headers — columns are A=name, B=division, C=role, D=email, E=phone. lookup-contact reads by 0-indexed position.
- **2026-04-24: Fallback is row index 0 of the actual Excel.** Earlier drafts hardcoded "row 1 = Hila" or "row 2 = Hila" but the real Excel varies. Fallback reads whoever is at row index 0.
- **2026-04-24: Topics 4 and 5 temporarily route to row 3 (Zohara Yishai).** Per `docs/nta-procedures.md`. Hila to confirm dedicated contacts before Phase 2.
- **2026-04-24: Working_hours field is hardcoded.** The Excel has no working_hours column. The contact card always shows `א-ה, 08:00-17:00`.
- **Demo runs on Telegram.** Phase 2 flips to WhatsApp via a single channel setting. Skills are channel-agnostic.
- **No persistence between sessions for the demo.** Memory clears after /end or 30-min idle. Each demo attendee gets a clean slate.
- **Hebrew is non-negotiable for user-facing text.** This repo's *.md files are English for developer readability only.
- **Procedural answers only from documented sources.** PE-100 and PE-060 facts live in `docs/nta-procedures.md`. Other procedures → route to Hila.
