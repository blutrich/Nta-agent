# Recovery Prompts

Quick fixes for the most common install failures.

---

## Recovery 1 — Agent is chatty / ignoring soul

Symptom: Agent says "בשמחה!" or "שאלה מצוינת!" or answers questions in free text.

```
Re-read soul.md. Your personality override didn't apply correctly.
You are a deterministic router. You do not chat. You do not answer questions outside the menu.
From now on: menu → selection → contact card. Nothing else.
Confirm by describing your job in 8 words or fewer.
```

---

## Recovery 2 — Skills missing from Files panel

Symptom: `find /app/.agents/skills -type f` returns fewer than 5 files, or returns errors.

```
The skills are missing from Files storage. Your previous write used bash and didn't persist.
Re-run Step 5 using write_file only — NO bash cat/cp/echo.
For each of the 5 skills (session-init, route-user, show-map, lookup-contact, handle-unknown):
call write_file with path=.agents/skills/{name}/SKILL.md and content=verbatim body from the repo.
Verify with find /app/.agents/skills -type f → expect exactly 5 SKILL.md files.
```

---

## Recovery 3 — Rules files not loading

Symptom: Agent doesn't know the menu Hebrew text, or doesn't enforce guardrails.

```
Re-run Step 4. For each of the 4 rules files, call write_file with path=.agents/rules/{filename}.
Files: menu-flows.md, guardrails.md, site-directory.md, allowlist.md.
Verify by opening the Files panel — all 4 must appear under .agents/rules/.
Do NOT use bash. write_file only.
```

---

## Recovery 4 — Telegram not connecting

Symptom: /start to the bot returns no response.

```
Check Settings → Channels → Telegram. Confirm the token is pasted correctly (no extra spaces).
If the token is correct, disconnect and reconnect.
If the bot username conflicts with an existing bot, create a new one via BotFather and use that token instead.
```

---

## Recovery 5 — Agent inventing contact details

Symptom: Phone numbers or emails in a contact card don't match Hila's Excel.

```
STOP. Do not send any more contact cards until this is resolved.
Re-read guardrails.md, specifically RULE G-1 and RULE G-2.
You must ONLY output contact details that appear verbatim in contacts.xlsx.
If the Excel row is empty, use the Hila fallback hardcoded in lookup-contact/SKILL.md.
Never generate a phone number or email from LLM knowledge.
Run the dry-run flow again (option 2 → site 10 → המשך) and show me the exact contact card output before sending to Telegram.
```

---

## Recovery 6 — Allowlist rejecting valid users

Symptom: Hila or test users get the rejection message even though their IDs were added.

```
Read allowlist.md from .agents/rules/. Show me the current PERMITTED_TELEGRAM_IDS list.
Compare against the 3 IDs I provided in setup.
If the IDs don't match, rewrite allowlist.md using write_file with the correct IDs.
Note: Telegram user IDs are integers, not strings. Make sure they're stored as integers.
```

---

## Recovery 7 — Map image not sending

Symptom: show-map skill runs but no image appears in Telegram.

```
Confirm petah-tikva-map.png is in Brain → Knowledge → Knowledge files.
If missing, re-upload the file.
If it's there but not rendering, the show-map skill should fall back to a text list of site numbers.
Update show-map/SKILL.md to confirm the fallback text is active.
```
