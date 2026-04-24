---
name: handle-unknown
description: Phase 5. Handles any input that doesn't match the expected flow — free text, bad site numbers, off-topic questions. Always re-prompts to menu or current step. Triggers on unrecognized input, free text during menu flow, off-topic questions.
---

# handle-unknown

Phase 5. The safety net. Catches anything that doesn't fit the decision tree.

## When To Run

- User sends free text when a button tap was expected
- User sends a site number that doesn't exist in the directory
- User sends text during map flow (instead of a number)
- User sends a question outside the 7 topics
- User sends an emoji, sticker, voice message, or file

## What It Does

1. Read `session.current_step` from Memory to determine context
2. Select the appropriate re-prompt message
3. Re-send the current step's keyboard or map
4. Do NOT attempt to answer the user's free-text question

## Re-Prompt By Context

### Context: main menu (current_step = null)

```
אנא בחר/י אחת מהאפשרויות בתפריט.
```

[re-send 7-option keyboard]

### Context: awaiting site number (current_step = "awaiting-site-number")

If the input is not a number:
```
אנא שלח/י את מספר האתר מהמפה (מספר בלבד).
```
[re-send map image]

If the input is a number but not in the directory:
```
לא מצאתי אתר במספר [N] במפה שלפניך.
אנא בדוק/י את מספר האתר ונסה/י שוב.
```
[re-send map image]

### Context: awaiting "המשך" tap (current_step = "awaiting-continue")

```
לחץ/י על הכפתור למטה לקבלת פרטי הקשר.
```
[re-send "המשך →" button]

### Context: any off-topic question (e.g., "מה שעות הפתיחה של נת"ע?")

```
לא מצאתי תשובה לשאלה זו בתפריט.
ניתן לפנות ישירות למנהלת אגף רשויות מקומיות דרך אפשרות 7.
```
[re-send 7-option keyboard]

## What NEVER To Do

- Never attempt to answer the free-text question using LLM knowledge
- Never say "I don't know" without a follow-up button
- Never leave the user without a keyboard or action to take
- Never treat a free-text message as intent classification ("I think you want option 2")

## Non-Text Message Types

Telegram sends stickers, voice messages, images, and files.
For all of these:

```
אני יכול/ה לקבל טקסט בלבד.
אנא בחר/י מהתפריט:
```
[re-send current context keyboard]

## Handling /menu Command

If user sends `/menu` at any point, clear current_flow and current_step, re-send the main 7-option menu. This is a manual escape hatch.

## Handling /end Command

```
תודה על הפנייה. נשמח לעזור שוב בכל עת.
לחץ /start להתחלה מחדש.
```

Clear all session state.
