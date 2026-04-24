# menu-flows.md

Authoritative Hebrew wording for every bot turn in the NTA Local Authorities Chat Agent.
This file is auto-loaded into the system prompt on every run via .agents/rules/.
The agent must use these exact strings — never paraphrase, never translate, never improvise.

---

## Main Menu

Sent after greeting. Always 7 inline keyboard buttons.

**Bot message:**
> איך אפשר לעזור לך היום?

**Buttons (in this exact order):**
1. אתר פעיל — עבודות נת"ע בביצוע
2. אתר עתידי — עבודות טרם החלו
3. תוכנית יזמית בגבולות תת"ל
4. סביבה ומיגון אקוסטי דירתי
5. נת"ע כרשות התמרור
6. הסברה לציבור ומפגשי תושבים
7. נושא אחר — דבר עם מישהו בנת"ע

---

## Flow 1 — אתר פעיל (עבודות בביצוע)

### Step 1 — Map prompt
**Bot message:**
> זוהי מפת האתרים הפעילים בעירך.  
> לאיזה מספר אתר תרצה לקבל מידע?

[send map image: petah-tikva-map.png]

### Step 2 — After user types site number (e.g., "2")
**Bot message:**
> ✅ ביקשת מידע על אתר [N] — [שם האתר].  
> האתר הינו חלק מקו המטרו [M2/M3].  
> לקבלת מידע על לוחות הזמנים, ניתן לפנות לאגף [שם האגף] בנת"ע.  
> לחץ **המשך** לקבלת פרטי הקשר.

**Button:** המשך →

### Step 3 — Contact card
[send contact card from lookup-contact skill]

---

## Flow 2 — אתר עתידי (עבודות טרם החלו)

### Step 1 — Map prompt
**Bot message:**
> זוהי מפת האתרים המתוכננים בעירך.  
> לאיזה מספר אתר תרצה לקבל מידע?

[send map image: petah-tikva-map.png]

### Step 2 — After user types site number
**Bot message:**
> ✅ ביקשת מידע על אתר [N] — [שם האתר].  
> האתר הינו חלק מקו המטרו [M2/M3].  
> לקבלת מידע על לוחות הזמנים המתוכננים, ניתן לפנות לאגף התכנון בקו [M2/M3].  
> לחץ **המשך** לקבלת פרטי הקשר.

**Button:** המשך →

### Step 3 — Contact card
[send contact card from lookup-contact skill]

---

## Flow 3 — תוכנית יזמית בגבולות תת"ל

**Bot message (single turn, no map):**
> ככלל, יש להגיש בקשה במערכת תיאום תשתיות.  
> לנת"ע יש 30 ימי בדיקה לכל בקשה שהוגשה במערכת.  
> במסגרתם, נת"ע יכולה לפנות ולבקש השלמות.  
>  
> אם עדיין נדרש בירור, ניתן לפנות לאגף תיאום תשתיות ותוכניות גובלות.  
> לחץ **המשך** לקבלת פרטי הקשר.

**Button:** המשך →

**Contact card:** row 3 from Excel (Infrastructure Coordination)

---

## Flow 4 — סביבה ומיגון אקוסטי דירתי

**Bot message (single turn):**
> לשאלות בנושאי סביבה, רעש ומיגון אקוסטי דירתי הקשורים לעבודות נת"ע, ניתן לפנות לאגף הסביבה.  
> לחץ **המשך** לקבלת פרטי הקשר.

**Button:** המשך →

**Contact card:** row from Excel (Environment Division) — if empty, fallback to row 2

---

## Flow 5 — נת"ע כרשות התמרור

**Bot message (single turn):**
> בסוגיות הקשורות להיותה של נת"ע רשות התמרור, ניתן לפנות לאגף התמרור.  
> לחץ **המשך** לקבלת פרטי הקשר.

**Button:** המשך →

**Contact card:** row from Excel (Traffic Signage) — if empty, fallback to row 2

---

## Flow 6 — הסברה לציבור ומפגשי תושבים

**Bot message (single turn):**
> לשאלות בנושאי הסברה לציבור, יידוע תושבים ומפגשי קהילה, ניתן לפנות לאגף התקשורת וההסברה.  
> לחץ **המשך** לקבלת פרטי הקשר.

**Button:** המשך →

**Contact card:** row from Excel (Communications) — if empty, fallback to row 2

---

## Flow 7 — נושא אחר

**Bot message:**
> תמיד אפשר לפנות בכל נושא למנהלת אגף רשויות מקומיות בנת"ע.  
> השותפות איתך חשובה לנו!  
> לחץ **המשך** לקבלת פרטי הקשר.

**Button:** המשך →

**Contact card:** row 2 from Excel (Hila Wechsberg, always)

---

## Contact Card Template

Sent after "המשך" is tapped. One message, copy-paste friendly.

```
📋 פרטי קשר — [שם האגף]

👤 [שם איש הקשר]
🏢 [תפקיד]
📞 [טלפון] — ניתן להתקשר ישירות
📧 [אימייל]
🕐 שעות פעילות: [שעות]
```

Followed by the "חזרה לתפריט" button.

**Button:** חזרה לתפריט הראשי ↩

---

## Error Messages

### Free text instead of button tap
> אנא בחר/י אחת מהאפשרויות בתפריט.

[re-send the 7-button keyboard]

### Site number not found
> לא מצאתי אתר במספר [N] במפה שלפניך.  
> אנא בדוק/י את מספר האתר ונסה/י שוב.

[re-send the map image]

### Excel row empty (fallback)
> לא מצאתי פרטי קשר ספציפיים לפנייה זו.  
> ניתן לפנות ישירות למנהלת אגף רשויות מקומיות:

[send Hila's contact card — row 2]

### Unlisted user
> הבוט נמצא בשלב פיילוט סגור.  
> לפרטים ניתן לפנות להילה וקסברג, מנהלת אגף רשויות מקומיות.  
> 📞 050-403-7303
