# site-directory.md

Site number → metro line → phase → Excel contact row lookup table.
Auto-loaded into system prompt via .agents/rules/.

This file covers Petah Tikva sites for the v0.1 demo.
In Phase 2, add a section per city, loaded based on session city.

---

## פתח תקווה — אתרים פעילים (זרימה 1) — עבודות AW בביצוע

| מס׳ אתר | שם האתר | קו | AW התחלה | AW סיום | row_id איש קשר | הערות |
|---------|---------|-----|---------|--------|--------------|-------|
| 1 | Kfar Ganim (LS) | M2 | 01/04/2026 | 01/09/2027 | 8 | טל מלכה — מנהל אגף הקמה M2 |
| 2 | Ben Gurion | M2 | 01/09/2026 | 01/09/2028 | 8 | טל מלכה — מנהל אגף הקמה M2 |
| 3 | Arlozorov | M2 | 01/05/2026 | 01/10/2027 | 8 | טל מלכה — מנהל אגף הקמה M2 |
| 4 | Petah Tikva City Hall | M2 | 01/09/2026 | 01/07/2027 | 8 | טל מלכה — מנהל אגף הקמה M2 |
| 5 | Petah Tikva East (LS) | M2 | 01/09/2026 | 01/09/2028 | 8 | טל מלכה — מנהל אגף הקמה M2 |
| 6 | Petah Tikva East | M2 | 01/05/2026 | 01/05/2027 | 8 | טל מלכה — מנהל אגף הקמה M2 |
| 7 | Segula Depot | M2 | 01/04/2025 | 31/12/2028 | 8 | טל מלכה — מנהל אגף הקמה M2 |

## פתח תקווה — אתרים עתידיים (זרימה 2) — אינפרא 1 טרם החל

| מס׳ אתר | שם האתר | קו | Infra1 התחלה | Infra1 סיום | row_id איש קשר | הערות |
|---------|---------|-----|------------|------------|--------------|-------|
| 10 | Sirkin South | M2 | 01/01/2029 | 01/09/2034 | 7 | דור נדל — מנהל מערך תכנון M2 |
| 11 | Sirkin North | M2 | 15/01/2029 | 01/09/2034 | 7 | דור נדל — מנהל מערך תכנון M2 |
| 12 | Segula Industrial Park | M2 | 15/01/2029 | 01/03/2033 | 7 | דור נדל — מנהל מערך תכנון M2 |
| 20 | Kfar Ganim (M3 Station) | M3 | 15/07/2028 | 20/02/2032 | 12 | רחלי ברגר — מנהלת אגף תכנון M3 |
| 21 | Soroka Belinson Hospital | M3 | 01/01/2029 | 30/07/2033 | 12 | רחלי ברגר — מנהלת אגף תכנון M3 |
| 22 | Kiryat Arye Industrial Area | M3 | 01/01/2029 | 30/03/2033 | 12 | רחלי ברגר — מנהלת אגף תכנון M3 |
| 23 | Kiryat Arye Stadium | M3 | 01/01/2029 | 30/12/2032 | 12 | רחלי ברגר — מנהלת אגף תכנון M3 |

---

## Lookup Logic

1. User sends site number (e.g., "21")
2. Agent checks current flow context (Flow 1 = active/AW, Flow 2 = future/Infra1)
3. Agent looks up this table for a matching row
4. If found → use the row_id to fetch contact from contacts.xlsx
5. If NOT found → send the "site not found" error message from menu-flows.md, re-send map

## Contact Row Reference (contacts.xlsx)

| row_id | Name | Role |
|--------|------|------|
| 1 | הילה וקסברג | מנהלת אגף רשויות מקומיות (Fallback) |
| 3 | זוהרה ישי | מנהלת אגף תיאום תשתיות ותוכניות גובלות |
| 6 | דקלה אסרף | קשרי קהילה M2 (public outreach M2) |
| 7 | דור נדל | מנהל מערך תכנון M2 (future M2 sites) |
| 8 | טל מלכה | מנהל אגף הקמה M2 (active M2 sites) |
| 9 | לאה שמול | מנהלת קו M3 |
| 11 | איציק | קשרי קהילה M3 (public outreach M3) |
| 12 | רחלי ברגר | מנהלת אגף תכנון M3 (future M3 sites) |
| 13 | שני שקד | מנהלת אגף הקמה M3 (active M3 sites) |

Source: contacts Excel provided by Hila Wechsberg (April 24, 2026) + Metro Gantt Stage A Final.
