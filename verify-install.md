# Verify Install

Run these tests after install to confirm the agent is working correctly before the demo.

---

## Test 1 — Skills Loaded

Send to the Superagent in chat:
```
Run: find /app/.agents/skills -type f
Expect exactly 5 SKILL.md files. List them.
```

Expected:
```
/app/.agents/skills/session-init/SKILL.md
/app/.agents/skills/route-user/SKILL.md
/app/.agents/skills/show-map/SKILL.md
/app/.agents/skills/lookup-contact/SKILL.md
/app/.agents/skills/handle-unknown/SKILL.md
```

If failed → Recovery 2

---

## Test 2 — Rules Loaded

```
What rules files do you have under .agents/rules/?
Quote the first line of guardrails.md.
```

Expected: Agent lists all 4 files and quotes "Hard rules enforced on every response."

If failed → Recovery 3

---

## Test 3 — Allowlist Check

Send from a Telegram account NOT on the allowlist:
```
/start
```

Expected: Rejection message in Hebrew. No menu shown.

Send from a Telegram account ON the allowlist:
```
/start
```

Expected: Greeting with first name, then 7-button keyboard.

If failed → Recovery 6

---

## Test 4 — Happy Path (Topic 2 → Site 10)

From an allowlisted Telegram account:

1. Send `/start`  
   → Expected: greeting + 7-button keyboard

2. Tap button 2 (אתר עתידי — עבודות טרם החלו)  
   → Expected: map prompt in Hebrew + petah-tikva-map.png image

3. Send "10"  
   → Expected: site 10 (בילינסון) confirmation + "המשך" button

4. Tap "המשך"  
   → Expected: contact card for M3 Planning Division (from Excel row 12)
   → Contact card must match Excel exactly — no invented details

5. Tap "חזרה לתפריט הראשי"  
   → Expected: 7-button keyboard reappears

Total time target: under 45 seconds.

If contact card has wrong data → Recovery 5

---

## Test 5 — Free Text Guardrail

From an allowlisted account, at the main menu:

Send: "מה שעות הפתיחה של נת"ע?"

Expected: re-prompt message + 7-button keyboard. NO free-text answer.

Send: "מה קורה עם קו M1?"

Expected: same re-prompt. NO speculation about metro lines.

If agent answers → Recovery 1

---

## Test 6 — Bad Site Number

During flow 2 (after map is sent):

Send: "99"

Expected: "לא מצאתי אתר במספר 99" + map image re-sent.

Send: "abc"

Expected: "אנא שלח/י את מספר האתר מהמפה (מספר בלבד)" + map image re-sent.

---

## Test 7 — Topic 7 Fallback

Tap button 7 (נושא אחר)

Expected: Hila's contact card (row 2 from Excel). No sub-menu, no further questions.

---

## Test 8 — Empty Excel Row Fallback

If topics 4, 5, or 6 have empty contact rows in the Excel:

Tap any of those buttons.

Expected: fallback contact card with Hila's details + the "לא מצאתי פרטי קשר" prefix message.

---

## Test 9 — Brain UI Registration

The most-missed install failure: skills and rules saved to disk but never registered in the Brain UI. The bot looks installed but doesn't actually use them.

```
Open Brain → Integrations → Skills. How many skills are listed?
Open Brain → Rules → Rules files. How many rules files are listed?
Run: find /app/.agents/skills -type f | wc -l
Run: find /app/.agents/rules -type f | wc -l
```

Expected:
- Brain Skills count = 5 (matches disk count)
- Brain Rules count = 4 (matches disk count)

If Brain count = 0 but disk count > 0 → **Recovery 8** in RECOVERY-PROMPTS.md (most common failure).

---

## Test 10 — End-Of-Install Summary Present

After Phase B autopilot finishes, the Phase C Summary message must include all 5 sections:

1. **What's installed** — 5 skills, 4 rules files, knowledge file counts, Brain UI counts
2. **What's connected** — Telegram bot username, allowlist count, tools permissions state
3. **What's verified** — dry-run results per scenario (happy path, free-text guardrail, bad site number, topic 7, contact card accuracy)
4. **What I can't do yet** — honest gaps (placeholder Telegram IDs, temp routing for topics 4-6, etc.)
5. **Next steps** — review date, demo date, Phase 2 trigger

If any section is missing or filled with `{placeholder}` text, send:

```
You forgot Phase C Summary or filled it with placeholders. Re-send it now with all 5 sections from BOOTSTRAP-PROMPT.md Phase C, using real data only. No "{X}" placeholders.
```

The Summary is the operator's only source of truth for what passed and what's gapped. Never ship an install without it.

---

## Common Issues

### "Bot responds but doesn't know the menu Hebrew text"

Rules files weren't registered in Brain — they're on disk but the runtime isn't loading them.
→ **Recovery 8** (Brain UI registration)

### "Bot answers free-text questions instead of re-prompting"

soul.md didn't override the default Base44 personality. The default is chatty.
→ **Recovery 1** (Agent is chatty / ignoring soul)

### "Contact card phone number doesn't match the Excel"

The agent is generating from LLM knowledge instead of reading the file. Critical guardrail break.
→ **Recovery 5** (Agent inventing contact details)

### "Allowlisted user gets the rejection message"

Telegram IDs in allowlist.md are stored as strings instead of integers, or the IDs don't match what the user actually has.
→ **Recovery 6** (Allowlist rejecting valid users)

### "Map image works in test but not in real Telegram"

petah-tikva-map.png isn't in Knowledge files, or Telegram is throttling image uploads.
→ **Recovery 7** (Map image not sending)

### "Bot responds normally but doesn't return to menu after contact card"

`current_flow` and `current_step` aren't being cleared after delivery. Check lookup-contact's "Session Reset After Delivery" section.

### "Two identical messages from one tap"

Telegram fired the same webhook twice. The session-init skill should deduplicate by `message_id`. If it's not, check the deduplication step.

### "Wrong site number returns the wrong contact"

site-directory.md has wrong row_id mappings. Verify against `docs/contacts-schema.md` Critical Row Mappings table — rows 7 (Dor Nadel, Planning) and 8 (Tal Malka, Execution) are easy to swap.

---

## Demo Readiness Checklist

Before April 27:

- [ ] All 10 tests pass
- [ ] Brain UI counts match disk counts (Test 9)
- [ ] Phase C Summary received and all 5 sections filled (Test 10)
- [ ] Full happy path (test 4) runs in under 45 seconds
- [ ] Contact card data verified by Hila against actual Excel
- [ ] 3 test Telegram users confirmed on allowlist
- [ ] QR code for bot link prepared for screen display
- [ ] Dry run with Hila completed (April 26)
- [ ] Backup: know which flows to demo if one fails (topics 2+7 are safest)
