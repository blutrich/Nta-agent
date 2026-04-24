# NTA Local Authorities Chat Agent

A Hebrew chat bot for Israel's 24 partner municipalities to reach the right NTA (Netivei Ayalon) division — without hunting through contact lists. Demo runs on Telegram; Phase 2 flips to WhatsApp.

Built as a Base44 Superagent configuration bundle.

## What it does

A rep from a partner city opens the bot, picks one of 7 topics, and gets back a verified NTA contact card (name, role, phone, email). For site-specific topics, the bot shows a city map, the user taps a site number, and the bot returns the right division for *that* site.

That's it. No small talk, no LLM-generated answers, no "I think you meant…". It's a router with a chat skin.

## Why it's built this way

Hila (head of Local Authorities at NTA) currently acts as a human switchboard — reps call her, she routes them. This bot does the same thing, 24/7, but only for the 7 topics she actually routes. Anything outside those 7 topics drops back to Hila.

The key design choice: **the LLM never improvises.** Every response is either pulled verbatim from Hila's Excel contacts file or from a pre-written Hebrew menu file. If the Excel lookup fails, the bot falls back to Hila's row — never a made-up phone number.

## The 7 topics

1. **Active site** — metro construction in progress (AW phase)
2. **Future site** — metro planning phase (Infra1)
3. **Development plans** bordering NTA infrastructure
4. **Environment & acoustic shielding**
5. **NTA as traffic signage authority** (רשות התמרור)
6. **Public outreach / resident meetings**
7. **Other** — forwarded to Hila

Topics 1 and 2 show a city map and ask for a site number. Topics 3–7 go straight to a contact card.

## The flow

```
/start
  ↓
Phase 1: session-init      → allowlist check, greet by name, show 7-button menu
  ↓
Phase 2: route-user        → parse selection (1-7), set flow state
  ↓
Phase 3: show-map          → (topics 1-2 only) send city map, parse site number
  ↓
Phase 4: lookup-contact    → Excel row → formatted contact card
  ↓
Phase 5: handle-unknown    → catches anything off-script, re-prompts to menu
```

Phases 1–5 live as skills under `skills/`. Each is one `SKILL.md` file — no code.

## Repo layout

```
Nta-agent/
├── CLAUDE.md                    # Context for AI agents working on the repo
├── BOOTSTRAP-PROMPT.md          # The install message Ofer pastes into a new Superagent
├── RECOVERY-PROMPTS.md          # Fix broken installs
├── verify-install.md            # Post-install test checklist
├── soul.md                      # Behavioral principles — be a router, not an AI
├── identity.md                  # Bot name, avatar, Hebrew tone, greeting template
├── rules/                       # Authoritative wording + data. Loaded into the system prompt.
│   ├── menu-flows.md            # Hebrew text for all 7 flows
│   ├── site-directory.md        # Site # → metro line → Excel row
│   ├── allowlist.md             # 3 permitted Telegram user IDs (demo)
│   └── guardrails.md            # Hard rules: never invent contacts
├── skills/                      # The 5 phases of the flow. Each is a SKILL.md.
│   ├── session-init/
│   ├── route-user/
│   ├── show-map/
│   ├── lookup-contact/
│   └── handle-unknown/
└── docs/                        # Reference material, not loaded as rules
    ├── contacts-schema.md       # How Hila's Excel maps to the bot
    ├── nta-procedures.md        # PE-100 / PE-060 procedural facts (topics 3, 4)
    └── phase2-whatsapp.md       # How to flip channel to WhatsApp after demo
```

## Deploying the demo (Ofer's flow)

1. Create a new Base44 Superagent
2. Paste `BOOTSTRAP-PROMPT.md` verbatim into the chat
3. Reply to the 4 setup questions:
   - Telegram bot token (from BotFather)
   - 3 Telegram user IDs for demo access
   - `contacts.xlsx` (upload)
   - `petah-tikva-map.png` (upload)
4. Wait for the 10 ✅ install markers
5. Run the `verify-install.md` checklist end-to-end
6. Share the Telegram bot link with Hila for review (April 26)
7. Live demo April 27

If anything breaks mid-install, see `RECOVERY-PROMPTS.md`.

## External dependencies

| What | Why it's needed | Required for demo |
|---|---|---|
| Base44 Superagent | Runtime | Yes |
| Telegram bot (BotFather) | Channel | Yes |
| `contacts.xlsx` | Source of truth for all contact cards | Yes |
| `petah-tikva-map.png` | Site map for topics 1 + 2 | Yes |
| WhatsApp (Base44 native) | Phase 2 channel | Phase 2 only |

## Gotchas for anyone editing this repo

- **Only `write_file` persists to Base44.** Bash `cat >`, `cp`, `echo >` all vanish on restart. If you add a skill or rule, it must be written through `write_file`.
- **Skills are folders, not loose files.** `skills/<name>/SKILL.md`. Never `skills/<name>.md`.
- **Disk ≠ Brain.** A skill on disk isn't a skill in Brain. Verify Brain → Integrations → Skills shows the right count after install. (See RECOVERY-PROMPTS.md Recovery 8 — most common failure.)
- **Hebrew keyboards on Telegram render RTL correctly** — but always verify on a real device on day one.
- **Telegram sometimes fires the same webhook twice.** `session-init` deduplicates by `message_id`.
- **Missing Excel rows fall back to Hila (row 1).** If topics 4–6 don't have dedicated rows yet, this is by design, not a bug.
- **Rows 7 and 8 are easy to swap.** Row 7 = Dor Nadel (M2 Planning, future sites). Row 8 = Tal Malka (M2 Execution, active sites).

## Phase 2

After the demo, the channel flips from Telegram to WhatsApp with a single setting in Base44. No skill or rule changes required. See `docs/phase2-whatsapp.md` for the migration plan.

The allowlist also changes from hardcoded Telegram IDs to phone-verified OTP against an NTA-maintained rep list.
