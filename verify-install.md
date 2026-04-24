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

## Demo Readiness Checklist

Before April 27:

- [ ] All 7 tests pass
- [ ] Full happy path (test 4) runs in under 45 seconds
- [ ] Contact card data verified by Hila against actual Excel
- [ ] 3 test Telegram users confirmed on allowlist
- [ ] QR code for bot link prepared for screen display
- [ ] Dry run with Hila completed (April 26)
- [ ] Backup: know which flows to demo if one fails (topics 2+7 are safest)
