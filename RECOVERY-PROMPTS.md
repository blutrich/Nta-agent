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
Check Settings → Channels → Telegram. Base44 has a native Telegram integration — no BotFather token required.
Disconnect and reconnect from Settings → Channels → Telegram.
If the connection still fails, check that no other Base44 agent on this account is already bound to the same Telegram bot.
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

---

## Recovery 8 — Skills/Rules on disk but NOT registered in Brain UI (most common Base44 install failure)

Symptom A: `find /app/.agents/skills -type f` shows 5 files, BUT Brain → Integrations → Skills tab shows 0 skills. Same for Rules: 4 files on disk, Brain → Rules tab shows 0.

Symptom B: Bot responds to /start but acts like a generic Base44 bot — doesn't know the menu, doesn't enforce guardrails, ignores soul.md. The skills exist on disk but the runtime isn't loading them.

**Root cause:** Base44 Superagents have TWO separate places skills/rules can live:
1. `.agents/skills/` and `.agents/rules/` folders on disk — easy for the agent to write
2. Brain → Integrations → Skills and Brain → Rules — what the runtime ACTUALLY uses

When told "create a skill", agents often just write a file without registering it in Brain. The skill exists but the agent can't call it.

```
The install is partially broken. Files may exist on disk under .agents/skills/ and .agents/rules/ but Brain → Integrations → Skills (and Brain → Rules) shows 0. They were never actually registered.

Fix this in exact order:

1. Run `ls .agents/skills/` and `ls .agents/rules/` and show me the output. Confirm files exist on disk.

2. Check Brain → Integrations → Skills count and Brain → Rules → Rules files count. Tell me both numbers.

3. REGISTER SKILLS IN BRAIN. Try these mechanisms in order until Brain UI count = 5:

   a. Chat-based custom skill creation. For each of the 5 skill files, send yourself:
      "Create a custom skill named 'session-init' with this body: [paste full file content]"
      Repeat for: session-init, route-user, show-map, lookup-contact, handle-unknown.

   b. If (a) doesn't update the UI, look for a built-in tool: create_skill, add_skill, register_skill, or install_skill. Call it for each skill.

   c. If neither works, use Base44's REST API: POST to whatever endpoint registers skills.

4. AFTER each registration attempt, verify the skill appears in Brain → Integrations → Skills (the UI tab). Don't trust file-on-disk presence. Verify the Brain UI count.

5. REPEAT FOR RULES FILES. The 4 files (menu-flows.md, guardrails.md, site-directory.md, allowlist.md) need to appear in Brain → Rules → Rules files. Use the chat-based rules upload command, then any registration tool, then the API.

6. Final verification:
   - Brain → Integrations → Skills count = 5
   - Brain → Rules → Rules files count = 4
   - .agents/skills/ disk count = 5
   - .agents/rules/ disk count = 4
   - Smoke test: `run_skill session-init` returns the SKILL.md body without "no executable script" errors

7. If Brain UI count still doesn't match disk count, tell me the SPECIFIC error and which Base44 mechanism failed. Do not say "done" until Brain UI shows the right counts.

Do not recreate files that already exist on disk. Just register them in Brain.
```

---

## Recovery 9 — Everything is broken, start over

Symptom: Too much is wrong to fix piece by piece. Multiple recoveries failed. Bot doesn't respond at all OR responds but with garbage.

**Option A: Rebuild skills + rules in place (keeps your Superagent, Soul, Identity, Telegram connection)**

```
Wipe .agents/skills/ and .agents/rules/ and start fresh. Keep everything else (Soul, Identity, Telegram, Knowledge files):

1. Delete all files under .agents/skills/ (if any)
2. Delete all files under .agents/rules/ (if any)
3. Re-clone the repo: https://github.com/blutrich/Nta-agent
4. Re-execute Steps 4 and 5 of BOOTSTRAP-PROMPT.md (install rules + skills) using write_file ONLY
5. Re-verify Brain UI counts match disk counts (per Recovery 8)
6. Run verify-install.md tests 1-4
7. Report results

My Soul, Identity, Telegram, and Knowledge files are already correct. Only skills and rules need rebuilding.
```

**Option B: Delete the Superagent and create a new one (nuclear)**

1. In Base44 → Superagents → your Superagent → Settings → Danger Zone → Delete this agent
2. Create a new Superagent
3. Re-paste the latest BOOTSTRAP-PROMPT.md from the repo
4. Re-upload the 2 files (contacts.xlsx, petah-tikva-map.png). Telegram reconnects via Base44's native integration — no token. Allowlist ships open.

Only do Option B if Option A fails repeatedly.

---

## After Recovery — Verify

Once you think the install is fixed, send:

```
Run verify-install.md tests 1, 2, 3, 4, and 7. Report pass/fail per test with specific evidence (skill counts, file paths, contact card field values). Do not say "all passed" without quoting the actual evidence.
```

If all 5 pass, the install is back online. Proceed to share the Telegram bot link with Hila.
