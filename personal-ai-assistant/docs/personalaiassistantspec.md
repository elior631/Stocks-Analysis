# מפרט ביצועי — עוזר AI אישי רב־מערכתי

> גרסה: 2.1
> תאריך עדכון: 12 ביולי 2026
> סטטוס: מאושר לביצוע — Step 0 הופעל
> קהל יעד: משתמש יחיד; לא מוצר SaaS ולא מערכת רב־משתמשית

### Changelog מגרסה 2.0 ל־2.1

- נוסף סעיף 13.5: חידודים ל־Capability Gap Memory (intent מובנה, Eval cases של הימנעות, סגירת מעגל בשחרור).
- נוסף סעיף 29: מדיניות Subagents — שלושה שימושים מותרים בלבד.
- נוסף סעיף 30: לולאת עלות/ביצועים למודלים — איך תמיד לדעת שנבחר המודל הנכון.
- סעיף "הפעולה הראשונה" הועבר לסוף כסעיף 31 ללא שינוי מהותי.

---

## 1. החלטת היסוד: להרכיב, לא לבנות פלטפורמה

המערכת היא כלי אישי עבור משתמש יחיד. לכן לא נבנה כעת תשתית של מוצר מסחרי: לא מערכת רב־דיירים, לא Control Panel מלא, לא מנוע אוטומציה חזותי, לא שכבת Adapters כללית ולא מערכת Agent עצמאית לכל שירות.

נשתמש בשירותים, SDKs, APIs ושרתי MCP קיימים כאשר הם בשלים ומתאימים למדיניות האבטחה. נכתוב קוד רק עבור החלקים הייחודיים:

1. מודל ההקשרים: Workspaces, Identities, Projects, Aliases ו־Connections.
2. מטריצת מדיניות והרשאות לפי חברה וזהות.
3. ניתוב פקודות בעברית והבנת זמן ישראלי.
4. כללי Jira, ובעיקר התאמת Project ו־Epic.
5. Validator דטרמיניסטי לפני ביצוע.
6. זיכרון תפעולי, Audit ו־Capability Gap Memory.
7. חוויית Telegram מהירה ומדויקת.

אם בעתיד יוחלט להפוך את הכלי למוצר, תיכתב ארכיטקטורה חדשה על בסיס נתוני שימוש אמיתיים. ה־MVP הנוכחי אינו צריך לשלם מראש את מחיר המורכבות של מוצר עתידי.

---

## 2. מטרת המוצר

ליצור נקודת כניסה אחת, מהירה ונוחה, שאליה אפשר לשלוח טקסט או הקלטה על כל תחום בחיים. העוזר יבין את ההקשר, יבחר את הזהות והמערכת הנכונות, יציע פעולה, יקבל אישור כשנדרש ויבצע אותה באופן בטוח וניתן למעקב.

### משפט מוצר

> אני שולח לעוזר מחשבה או פקודה בשפה טבעית; הוא מבין לאן היא שייכת, מציע את הפעולה הנכונה ומבצע אותה עם מינימום חיכוך ומקסימום שליטה.

### הצלחה נמדדת לפי

- פחות פעולות ידניות ומעברים בין אפליקציות.
- אפס פעולות בחברה או בזהות הלא נכונות.
- משימה או תזכורת לא רק נשמרת, אלא גם מגיעה בזמן.
- כל פעולה חיצונית ניתנת להסבר, מעקב וביטול כאשר הדבר אפשרי.
- פקודות שהמערכת אינה יודעת לבצע הופכות ל־Backlog מדיד לשדרוג הבא.

---

## 3. מה המוצר אינו

- הוא אינו Second Brain ואינו מחליף כרגע Obsidian, Notion או מערכת ניהול ידע.
- הוא אינו מקור האמת של Jira, Google Tasks, Gmail או Calendar.
- הוא אינו שולח הודעות או מיילים ללא מדיניות ואישור מתאימים.
- הוא אינו בוט אוטונומי שמחליט אילו כלים להפעיל ללא גבולות.
- הוא אינו מערכת Voice בזמן אמת ב־MVP.
- הוא אינו משתמש ב־Vector DB לפני שקיימות בעיות Retrieval מוכחות.

---

## 4. עקרונות מחייבים

### 4.1 Planner proposes, code validates, tools execute

מודל השפה מציע `ActionPlan`. קוד דטרמיניסטי בודק אותו מול ההקשר, ההרשאות, הנתונים והמדיניות. רק לאחר מכן כלי חיצוני מבצע.

### 4.2 זהות היא גבול אבטחה

כל חיבור קשור ל־Identity ול־Workspace יחידים. אין Credential כללי לכל החברות. המודל אינו מקבל Tokens או סיסמאות; הוא מחזיר `identity_id` בלבד.

### 4.3 מקור האמת נשאר ביעד

Ticket נשמר ב־Jira, משימה ב־Google Tasks, אירוע ב־Calendar ומייל ב־Gmail. Postgres שומר Metadata, Audit, מיפויים וזיכרון תפעולי.

### 4.4 Shadow Mode קודם לביצוע

לפני חיבור פעולות כתיבה, המערכת מריצה פקודות אמיתיות, מציגה מה הייתה עושה ושומרת תיקונים כ־Eval cases.

### 4.5 אין Ticket ללא Epic — אלא אם הוגדר חריג

ברירת המחדל: `create_jira_issue` נכשל אם אין Parent Epic מאומת. ניתן להגדיר `epic_required: false` לפרויקט מסוים בלבד.

### 4.6 Telegram אינו ערוץ סודי

הודעות Bot רגילות אינן End-to-End encrypted ונשמרות בתשתית Telegram. לכל Workspace יוגדר `telegram_allowed` ו־`confidentiality_level`. בחברה האוסרת שימוש כזה, Telegram יקבל רק ניסוח לא רגיש או שה־Workspace ייחסם לחלוטין.

### 4.7 Idempotency לפני Retry

כל פעולה חיצונית מקבלת `idempotency_key`. Retry לא ייצור Ticket, משימה או הודעה כפולים.

### 4.8 מודלים וכלים ניתנים להחלפה

הקוד פונה ל־Provider interface קטן. בחירת מודל או MCP אינה חודרת למודל הנתונים או למדיניות.

### 4.9 Subagents רק כשהם קונים בידוד

תת־סוכן (קריאת מודל נפרדת עם הקשר/כלים משלה) מוצדק רק כשהוא קונה בידוד של הקשר, הרשאות או סיכון — לא כדי "לחלק עבודה" בפקודה רגילה. הפירוט המלא בסעיף 29.

---

## 5. היקף V1

V1 כולל:

- Telegram פרטי: טקסט, Voice, כפתורי אישור וביטול.
- ניתוב Workspace / Project / Identity.
- Shadow Mode ואיסוף Eval אוטומטי.
- Google Tasks: יצירה, עדכון והשלמה.
- תזכורות אמיתיות דרך Telegram scheduler.
- Collections פנימיות ב־Postgres.
- Jira עבור חברה אחת, עם Project ו־Epic מאומתים.
- Audit query: חיפוש פעולות שבוצעו.
- Capability Gap Memory לפקודות לא נתמכות.
- גיבוי וייצוא CSV/JSON.
- בדיקת תקינות חיבורים יומית.

V1 אינו כולל:

- Mini App או Control Panel מלא.
- Gmail send אוטומטי.
- WhatsApp Business API.
- Calendar write.
- Agent אוטונומי ארוך־טווח.
- Vector DB.
- Multi-user או הרשאות מנהלים.

---

## 6. הכלים שנבחרו

| שכבה | בחירה | סיבה |
|---|---|---|
| ממשק | Telegram Bot + grammY | UX מהיר, טקסט/Voice/Buttons, ספריית TypeScript בשלה |
| Runtime | Node.js 22 + TypeScript | טיפוסים, SDKs טובים, בדיקות ותחזוקה פשוטה |
| שירות | Fastify או Hono | Webhooks, OAuth callbacks ו־health endpoints בשירות קטן |
| Agent harness | Vercel AI SDK או SDK ספק דק | Structured Output ו־tool calling בלי לבנות Agent framework |
| מסד נתונים | Supabase Postgres | Postgres מנוהל, migrations, גיבוי ושדרוג פשוט |
| Deploy | Railway / Fly.io / Render, לפי בדיקת מחיר בזמן ההקמה | שירות יחיד קטן ו־scheduler |
| תורים | Postgres jobs תחילה | אין צורך ב־Redis או Queue חיצוני ב־V1 |
| Scheduler | Cron בשירות + נעילת Postgres | תזכורות, health checks ו־daily brief |
| Jira | REST API ישיר עם API token ב־MVP; Rovo MCP אופציונלי | API token פשוט למשתמש יחיד; MCP רק אם הרשאות ומדיניות מתאימות |
| Google Tasks | Google Tasks API רשמי | Surface קטן ופשוט; אין צורך ב־MCP כללי |
| Gmail/Calendar | APIs רשמיים, כשייכנסו לגרסה | שליטה מדויקת ב־scopes ובהרשאות |
| סודות | Secret manager של פלטפורמת האירוח | אין סודות ב־Git, YAML או prompt |
| קונפיגורציה | YAML פרטי ב־Git + טבלאות Runtime | Versioning, diff ו־rollback בלי לבנות UI |
| בדיקות | Vitest + Eval runner ייעודי | Unit tests וכלי השוואת מודלים על פקודות אמיתיות |
| ניטור | Structured logs + Sentry אופציונלי | מספיק למערכת אישית; אין צורך ב־observability stack כבד |

### למה n8n הוסר

n8n לא יהיה חלק מהארכיטקטורה הראשית. במקרה הזה הוא יגרום לבניית state machine פעם אחת כ־Workflow ופעם נוספת בקוד כאשר הלוגיקה תגדל. שירות TypeScript יחיד זול יותר, ניתן לבדיקה ומבטל תלות של כ־€20 לחודש.

### למה אין Mini App

הקונפיגורציה תנוהל תחילה בקובצי YAML ובפקודות Telegram. OAuth דורש דף Callback קטן בלבד. UI ייבנה רק אם ניהול שיחתי ו־YAML יוכחו כמגבלה אמיתית.

### תפקיד MCP

MCP הוא אמצעי חיבור, לא הארכיטקטורה. נשתמש בשרת רשמי כאשר הוא:

- נתמך על ידי ספק המערכת.
- מאפשר הרשאות מינימליות.
- מספק Audit ו־OAuth מתאימים.
- אינו מטשטש את ה־Identity או ה־Workspace.
- אינו מוסיף יכולות רחבות יותר מהנדרש.

אם התנאים אינם מתקיימים, API רשמי וישיר עדיף. Atlassian Rovo MCP הוא מועמד רלוונטי; Google Tasks API ישיר פשוט יותר ל־V1.

---

## 7. ארכיטקטורה

```text
Telegram / Future Email Ingress
              │
              ▼
      Ingestion + State Machine
              │
       ┌──────┴──────┐
       ▼             ▼
 Transcription   Context Retrieval
       └──────┬──────┘
              ▼
        LLM Action Planner
              │ ActionPlan JSON
              ▼
 Deterministic Validator + Policy Matrix
              │
       ┌──────┴─────────┐
       ▼                ▼
 Clarify / Approve   Capability Gap
       │                Memory
       ▼
  Tool Executor ──► Jira / Google / Postgres
       │
       ▼
 Audit + Scheduler + Eval Flywheel
```

זהו שירות TypeScript אחד ו־Postgres אחד. ההפרדה היא לוגית ומודולרית, לא אוסף Microservices.

### מודולים בקוד

```text
src/
├── bot/
├── conversation/
├── context/
├── planner/
├── validation/
├── policy/
├── capabilities/
├── tools/
│   ├── jira/
│   ├── google-tasks/
│   └── collections/
├── reminders/
├── health/
├── audit/
└── evals/
```

אין ליצור interfaces כלליים מעבר למה שנדרש בפועל. כל abstraction צריך לפחות שני consumers אמיתיים או סיבה בטיחותית ברורה.

---

## 8. מודל הקשרים וקונפיגורציה

### Workspace

עולם תפעולי מבודד: חברה, פרויקט שותפים או אישי.

```yaml
id: company_a
name: Company A
type: company
confidentiality_level: internal
telegram_allowed: true
default_timezone: Asia/Jerusalem
default_identity: company_a_google
```

### Identity

```yaml
id: company_a_google
workspace: company_a
email: me@company-a.com
provider: google
connection_ref: google_company_a
allowed_actions:
  - tasks.read
  - tasks.write
```

### Project

```yaml
id: mobile_app
workspace: company_a
aliases: ["האפליקציה", "מובייל", "הפרויקט של דנה"]
jira:
  site: company-a.atlassian.net
  project_key: MOB
  epic_required: true
  default_issue_type: Task
```

### Policies

```yaml
workspace: company_a
actions:
  jira.create:
    approval: always
  gmail.send:
    approval: always
  tasks.create:
    approval: first_20_then_optional
```

### הוספת חברה, פרויקט או מייל

ב־MVP הבוט ינהל Wizard שיחתי ויציע Pull Request/commit לקובצי YAML. המשתמש מאשר את השינוי. Credentials לעולם אינם נכתבים ל־YAML; חיבור חשבון נעשה בדף OAuth קטן או מוזן כ־secret דרך פלטפורמת האירוח.

---

## 9. Conversation State Machine

לכל Chat יש מצב מפורש:

- `idle`
- `awaiting_clarification`
- `awaiting_approval`
- `executing`
- `completed`
- `cancelled`
- `failed`

### כללי הפרעה

כאשר ממתינה הבהרה או אישור ומגיעה הודעה חדשה, המערכת לא תניח אוטומטית שזו תשובה. היא תציג:

1. לענות לבקשה הממתינה.
2. להתחיל פקודה חדשה ולהשאיר את הקודמת מושהית.
3. לבטל את הבקשה הקודמת.

תשובות קצרות כמו "כן", "לא", שם חברה או מספר אפשרות ישויכו לבקשה ממתינה רק אם הן מתאימות ל־expected reply schema.

לכל בקשה ממתינה יש TTL. לאחר פקיעתו היא עוברת ל־`cancelled` ואינה ניתנת לביצוע בטעות.

---

## 10. ActionPlan

המודל מחויב ל־Structured Output:

```json
{
  "request_id": "uuid",
  "intent": "create_jira_issue",
  "workspace_id": "company_a",
  "identity_id": "company_a_atlassian",
  "project_id": "mobile_app",
  "parameters": {
    "summary": "תקלה בתהליך ההתחברות",
    "epic_key": "MOB-120",
    "due_at": null
  },
  "confidence": {
    "workspace": 0.98,
    "identity": 0.99,
    "project": 0.93,
    "action": 0.97
  },
  "requires_approval": true,
  "missing_fields": [],
  "unsupported_reason": null
}
```

Schema-valid JSON אינו מספיק לבטיחות. ה־Validator בודק סמנטיקה והרשאות:

- Workspace קיים ופעיל.
- Identity שייך ל־Workspace.
- Connection תקין.
- Project שייך ל־Workspace.
- Epic קיים, פתוח ומתאים לפרויקט.
- פעולה מותרת במדיניות.
- תאריך תקין באזור הזמן הנכון.
- אין Idempotency key שכבר בוצע.
- אין Duplicate סביר בטווח המוגדר.

---

## 11. Jira

### אימות לפני יצירה

1. Resolver בוחר Workspace, Project ו־Identity.
2. Metadata נשלף מה־cache ומתעדכן לפי TTL.
3. Epic נפתר לפי key, alias או חיפוש מוגבל.
4. Validator בודק Project, Issue Type, Parent ושדות חובה.
5. המשתמש רואה Preview ומאשר.
6. הפעולה נוצרת עם idempotency key.
7. נשלח קישור ל־Issue מיד.
8. אימות הופעה ב־Board filter מתבצע אסינכרונית.
9. המשתמש מקבל הודעה נוספת רק אם קיימת אי־התאמה.

### Duplicate Detection מוגבל

הבדיקה תכלול רק:

- `action_runs` של המשתמש מהזמן האחרון.
- Issues פתוחים תחת ה־Epic המועמד.
- דמיון בכותרת בטווח זמן מוגדר.

לא יתבצע חיפוש JQL רחב לפני כל יצירה.

### Auth

ב־MVP משתמשים ב־API token נפרד לכל אתר/זהות, שמור כ־secret מוצפן. OAuth 3LO יישקל רק אם מדיניות החברה מחייבת או אם המערכת תהפוך לרב־משתמשית.

---

## 12. Google Tasks ותזכורות אמיתיות

Google Tasks הוא מקור האמת למשימות אישיות, אך העוזר אחראי להבטחת התזכורת.

כאשר מתקבלת פקודה כמו "תזכיר לי ביום ראשון בערב":

1. Relative Date Resolver דטרמיניסטי מתרגם את הביטוי לזמן מוחלט.
2. המודל מציע שעה אם היא חסרה, אך אינו קובע לבדו.
3. המשתמש רואה את התאריך והשעה המפורשים.
4. נוצרת Google Task.
5. נוצרת רשומת `scheduled_notifications` מקומית.
6. Scheduler שולח Telegram push בזמן.
7. נרשמת מסירה או שגיאה ב־Audit.

### זמן עברי וישראלי

- ברירת המחדל: `Asia/Jerusalem`.
- שבוע ישראלי מתחיל ביום ראשון.
- ביטויים כמו "מחר בערב" ממופים לפי טבלת כללים ניתנת להגדרה.
- DST נפתר באמצעות ספריית זמן תקנית ולא באמצעות חישוב ידני.
- כל זמן יחסי מוצג כאישור מוחלט לפני שמירה.

---

## 13. Capability Gap Memory

כאשר המשתמש מבקש פעולה שהעוזר אינו יודע לבצע, אסור להמציא כלי או להעמיד פנים שהפעולה בוצעה.

### התנהגות למשתמש

העוזר ישיב:

> אני עדיין לא יודע לבצע את הפעולה הזו. שמרתי אותה כהצעת יכולת. כרגע אני יכול להציע דרך ידנית או להכין טיוטה.

אם אפשר, יוצעו:

- Workaround ידני.
- טיוטה או נתונים מוכנים להעתקה.
- כלי או API אפשריים.
- רמת סיכון והרשאות צפויות.

### מה נשמר

טבלת `capability_gaps`:

```text
id
created_at
original_request_redacted
normalized_intent
workspace_id
category
desired_outcome
missing_capability
suggested_tool_or_integration
workaround
frequency
last_requested_at
estimated_value
estimated_effort
risk_level
required_permissions
status
linked_release
```

### פרטיות

יישמר ניסוח מצונזר ככל האפשר. Secrets, תוכן סודי ופרטים שאינם נחוצים להבנת היכולת לא ייכנסו ל־Backlog.

### מנגנון המלצה לגרסה הבאה

פעם בשבוע או לפי פקודה `/gaps`, המערכת תקבץ בקשות דומות ותדרג אותן:

```text
upgrade_score = frequency × time_saved × strategic_fit × confidence
                ─────────────────────────────────────────────────
                         implementation_effort × risk
```

הדוח יציג:

- חמש היכולות המבוקשות ביותר.
- כמה פעמים כל אחת התבקשה.
- Workaround קיים.
- כלי או API מומלץ.
- Scopes והרשאות נדרשים.
- הערכת מאמץ ועלות.
- סיכון פרטיות ואבטחה.
- הצעה לאיזו גרסה לשייך.

שדרוג אינו מתבצע אוטומטית. המשתמש מאשר הכנסת יכולת ל־Roadmap, ולאחר פיתוח היא מקבלת capability id ו־Eval cases מתוך הבקשות המקוריות.

### 13.5 חידודים

1. **הזיהוי הוא חלק מה־Schema, לא מקרה קצה.** ל־ActionPlan יש `intent: "unsupported_request"` מפורש עם `desired_outcome` ו־`missing_capability`. בלי intent ייעודי המודל עלול "למתוח" בקשה לא נתמכת לתוך intent קיים — טעות מסוכנת יותר מסירוב מפורש.
2. **כל gap הופך גם ל־Eval case מסוג הימנעות.** מלבד רישום ב־Roadmap, נוצר eval case שמוודא שגרסאות עתידיות של הסוכן ממשיכות לסרב נכון לפעולה, עד שהיא תפותח בפועל. כך הרגרסיה נבדקת גם על מה שהמערכת *לא* אמורה לעשות.
3. **סגירת מעגל בשחרור.** כשיכולת מפותחת, כל הבקשות המקוריות שהובילו אליה הופכות ל־Eval cases חיוביים של אותה יכולת, וה־gap מסומן `linked_release`. כל דור חדש של הסוכן נבחן במפורש על מה שביקשת ולא קיבלת בדור הקודם.

---

## 14. Shadow Mode ו־Eval Flywheel

כל תיקון ב־Shadow Mode הופך אוטומטית ל־Eval case:

```json
{
  "input": "תפתח טיקט על הבאג באפליקציה",
  "expected": {
    "workspace_id": "company_a",
    "project_id": "mobile_app",
    "intent": "create_jira_issue"
  },
  "forbidden": ["execute_without_epic"],
  "source": "shadow_correction"
}
```

### Dataset ראשון

לפני בניית האינטגרציות ייאספו לפחות 100 פקודות אמיתיות:

- 30 פקודות חד־משמעיות.
- 20 פקודות עמומות.
- 15 פקודות מרובות פעולות.
- 15 פקודות זמן ותזכורת.
- 10 פקודות Jira/Epic.
- 10 פקודות לא נתמכות או מסוכנות.

### תנאי יציאה מ־Shadow Mode

נדרשות לפחות 200 הרצות מצטברות, כולל 100 פקודות אמיתיות שונות:

- 100% Schema validity.
- לפחות 95% Workspace accuracy.
- לפחות 95% Intent accuracy.
- 100% חסימה של Identity שאינו שייך ל־Workspace.
- 100% חסימה של Jira ללא Epic בפרויקט המחייב Epic.
- אפס ביצועי כתיבה בעולם הלא נכון.
- לפחות 95% דיוק בפתרון תאריכים יחסיים בסט הבדיקות.

גם לאחר היציאה, פעולות תקשורת ו־Jira נשארות עם אישור מפורש.

---

## 15. בחירת מודלים

הבחירה הסופית תיעשה לפי ה־Eval בעברית, לא לפי מותג. ברירת המחדל הראשונה נועדה להתחיל מהר ולצמצם אינטגרציות. העיקרון המנחה: הארכיטקטורה (Validator + Policy + אישור) קונה את הבטיחות, ולכן המודל עצמו לא צריך להיות פרימיום בכל שלב — משלמים על מודל חזק רק בנקודה הצרה שבה טעות באמת יקרה. ראו לולאת ההחלטה המלאה בסעיף 30.

### Planner ראשי ל־V0/V1: GPT-5.4 mini

שימושים:

- ניתוח פקודה בעברית.
- Structured ActionPlan.
- פירוק הודעה למספר פעולות.
- שאלת הבהרה קצרה.
- Capability gap classification.

סיבות:

- Structured Outputs ו־tool calling בשלים.
- יחס עלות/אמינות מתאים לפקודות מרובות הקשר.
- עלות המודל זניחה יחסית לזמן פיתוח ולטעות בזהות.
- Provider יחיד ב־MVP מפחית תקלות.

מחיר בסיס בזמן כתיבת המסמך: $0.75 למיליון טוקני קלט ו־$4.50 למיליון טוקני פלט. יש לאמת מחיר לפני פתיחת Billing.

### תמלול: GPT-4o mini Transcribe

ברירת המחדל הראשונה: כ־$0.003 לדקה. הוא ייבדק מול דגימות עברית, שמות אנשים ושמות חברות.

חלופות כמו Deepgram, Groq או מודלים מותאמים לעברית (למשל משפחת ivrit.ai על בסיס Whisper) ייבחנו רק על אותו corpus. לא נחליף ספק על בסיס מחיר בלבד; שיפור בזיהוי שמות שווה יותר מחיסכון של דולרים בודדים — דיוק שמות הוא צוואר הבקבוק שממנו תלוי כל ניתוב ההקשר שאחריו.

### מועמדי Benchmark

- Muse Spark 1.1: מועמד ל־planner/tool use, אך לא ברירת מחדל לפני בדיקת עברית, יציבות וזמינות.
- Gemini Flash-Lite: מועמד לניתובים פשוטים וזולים, רק לאחר שנתוני שימוש מראים נפח משמעותי של פקודות טריוויאליות.
- מודל חזק יותר (Critic/Verifier): fallback לפעולות עתירות־סיכון בטווח confidence בינוני, לא ברירת מחדל לכל פקודה. פירוט בסעיף 29.2.

### Prompt Caching

System prompt, schema ומטא־דאטה יציב יופרדו מתוכן המשתמש ויישלחו באופן שתומך caching. Metadata יישלף לפי המועמדים הרלוונטיים בלבד ולא כל רשימת החברות והפרויקטים.

---

## 16. Latency Budget

| אירוע | יעד |
|---|---:|
| ACK ב־Telegram | פחות משנייה |
| פקודת טקסט עד Preview | עד 6 שניות ב־p95 |
| Voice קצר עד Preview | עד 8 שניות ב־p95 |
| פעולה מאושרת עד אישור קבלה | עד 2 שניות |
| Jira create עד קישור | עד 6 שניות ב־p95 |

הבוט שולח ACK מיד. לאחר Voice, אחזור Context שאינו תלוי בתמלול מתחיל במקביל. Board verification, Audit enrichment ו־Eval generation מתבצעים אחרי התגובה למשתמש.

---

## 17. Google OAuth והרשאות

לכל חשבון Google Connection נפרד. יש לתעד:

- OAuth consent status.
- Scopes מדויקים.
- תאריך הוצאת refresh token.
- האם החשבון Personal או Workspace.
- האם האפליקציה במצב Testing, Internal או Production.
- תאריך בדיקת החיבור האחרונה.

Refresh tokens באפליקציית OAuth במצב Testing עלולים לפוג לאחר שבעה ימים. לכן לפני חיבור חשבונות אמיתיים מחליטים אם לפרסם את האפליקציה ל־Production לשימוש אישי, או להשתמש ב־Internal בחשבון Workspace מתאים.

`gmail.send` אינו נכנס ל־V1. לפני הוספתו בודקים את סיווג ה־scope, דרישות האימות ומדיניות כל חברה. Draft-only יועדף לפני Send.

---

## 18. Credential Health

Job יומי בודק כל Connection באמצעות קריאה זולה ולא הרסנית:

- Google token refresh או API ping.
- Jira `myself`/project metadata ping.
- Telegram webhook health.
- מצב Scheduler והודעה אחרונה.

המשתמש מקבל התראה לפני פעולה דחופה:

> החיבור ל־Google Tasks של Company A אינו תקין. נדרש חיבור מחדש.

אין לבצע Retry אינסופי. כשחיבור פג, פעולות חדשות נשמרות כ־pending ואינן מועברות לזהות אחרת.

---

## 19. Audit, חיפוש וגיבוי

כל פעולה נשמרת ב־`action_runs` עם:

- בקשה מצונזרת.
- ActionPlan.
- גרסת prompt ומודל.
- Workspace, Identity ו־Project.
- אישור המשתמש.
- tool input מצונזר.
- תוצאה, זמן, latency ועלות משוערת.
- idempotency key.

הפקודה `/find` מאפשרת שאלות כגון:

- "מה הטיקט שפתחתי על ההרשאות?"
- "מה עשיתי השבוע?"
- "אילו תזכורות נכשלו?"

### גיבוי וייצוא

- גיבוי אוטומטי של Postgres לפי יכולות התוכנית.
- Export ידני ל־CSV/JSON עבור Collections, capability gaps ו־action history.
- Dump מוצפן תקופתי לאחסון נפרד לאחר מעבר לשימוש יומיומי.
- בדיקת Restore רבעונית; גיבוי שלא נבדק אינו נחשב גיבוי.

---

## 20. Rate Limits ו־Quotas

לפני הפעלת כל Adapter תתועד טבלה עם:

| מערכת | מגבלה/Quota | אסטרטגיה |
|---|---|---|
| Telegram | מגבלות שליחה לפי Bot/chat | batching ו־backoff |
| Google Tasks | quota לפי פרויקט/משתמש | cache, retry with jitter |
| Gmail | daily send limits + scopes | staged send, לא ב־V1 |
| Jira | rate limiting לפי cloud/site | bounded queries, backoff |
| Model API | TPM/RPM | queue קצר ו־fallback ברור |

הערכים עצמם ייקראו מהחשבונות והמסמכים הרשמיים בזמן ההקמה, משום שהם תלויים בתוכנית ובארגון.

---

## 21. תכונות לפי סדר ערך

### V1 — Execution Core

- Telegram טקסט/Voice.
- Shadow Mode + Eval flywheel.
- Google Tasks + Telegram reminders.
- Collections.
- Jira חברה אחת.
- `/find` על Audit.
- Capability Gap Memory.
- Credential health.
- Export.

### V1.1 — Daily Utility

- Morning Brief: משימות להיום, תזכורות, approvals תקועים ו־connection warnings.
- Commitment tracking: "אם שרון לא ענתה עד שלישי, תזכיר לי".
- Staged external send: השהיה של 60 שניות וכפתור Cancel.
- Templates/macros לפעולות חוזרות.

### V1.2 — Inbound Email Triage

- Gmail label ייעודי או Forward address.
- סיווג: Ticket / Task / Reply Draft / Archive reference.
- ברירת מחדל Draft-only.
- אישור לפני יצירה או שליחה.
- טיפול ב־prompt injection מתוך תוכן מייל כ־untrusted input, דרך Subagent הסגר (סעיף 29.1).

### V1.3 — Batch and Review

- Paste של סיכום פגישה.
- N פעולות מוצעות במסך Telegram קומפקטי.
- Approve all / approve selected / edit.
- Daily Brief ושדרוג Weekly Review.

### V2 — Multi-Company

- הוספת Connections נוספים אחד־אחד.
- מדיניות סודיות לכל Workspace.
- Jira נוסף רק לאחר מעבר Eval ייעודי.
- Calendar read-only.
- WhatsApp click-to-chat.

### V3 — Communication and Proactivity

- Gmail Draft ולאחר מכן Send, בכפוף למדיניות OAuth.
- Calendar write.
- WhatsApp Business API רק אם יש הצדקה עסקית.
- Learned automation proposals; לעולם לא הפעלה עצמית ללא אישור.

### לא מתוכנן עד שיש כאב מוכח

- Mini App מלא.
- Vector DB.
- Realtime voice.
- Microservices.
- מערכת Multi-user.

---

## 22. עלויות

המחירים משתנים ויש לאמת אותם ביום ההקמה. ההערכה מיועדת לתכנון, לא להצעת מחיר.

### MVP רזה

| רכיב | עלות חודשית צפויה |
|---|---:|
| Telegram Bot | $0 |
| GitHub private repo | $0 בתוכנית מתאימה |
| Supabase Free | $0 |
| Hosting לשירות TypeScript קטן | בערך $5–10 |
| GPT-5.4 mini | בערך $2–8 |
| תמלול אישי | בערך $0.50–2 |
| Sentry/ניטור בסיסי | $0 בתחילה |
| סך קבוע ומשתנה | בערך $8–20 |

### לאחר שהמערכת קריטית

- Supabase Pro: החל מכ־$25 לחודש.
- Hosting עם גיבוי/זמינות טובים יותר: בערך $10–25.
- מודלים ותמלול: בדרך כלל $5–15 בשימוש אישי.
- סך סביר: כ־$40–65 לחודש.

עלות הפיתוח היא העלות העיקרית. לכן לא מוסיפים ספק, UI או שירות תשתית לפני שקיים כשל מדיד שהם פותרים.

### דוגמת עלות מודל

1,500 פקודות בחודש, עם 1,500 טוקני קלט ו־250 טוקני פלט בממוצע, הן כ־$3.38 ב־GPT-5.4 mini לפני caching. עשר דקות Voice ביום הן כ־$0.90 בחודש ב־GPT-4o mini Transcribe. אלו הערכות בלבד.

---

## 23. סדר עבודה — צעד אחר צעד

### שלב 0 — יום 1: Repository והחלטות

1. לפתוח Repository פרטי.
2. להוסיף את המסמך הזה.
3. ליצור `config/`, `evals/`, `src/`, `supabase/` ו־`docs/`.
4. לקבוע Budget alert לספק המודל.
5. להגדיר שאין Production credentials בשלב זה.

תוצר: שלד מתועד, ללא חיבורים חיצוניים.

### שלב 1 — ימים 1–3: Eval לפני תשתית

1. לאסוף 100 פקודות אמיתיות בעברית.
2. לתייג expected workspace, intent, project, identity ו־date.
3. להוסיף negative cases ופקודות לא נתמכות.
4. להריץ GPT-5.4 mini ולשמור תוצאות.
5. להריץ Muse/Gemini על אותו סט רק אם הגישה אליהם פשוטה.
6. לבחור לפי accuracy, latency ועלות.

תוצר: baseline מדיד. אין לבחור מודל מחדש לפי תחושת בטן.

### שלב 2 — ימים 3–5: בסיס הנתונים והקונפיגורציה

1. לפתוח Supabase Free.
2. ליצור migrations לטבלאות הליבה.
3. לכתוב 3–5 Workspaces אמיתיים ב־YAML, בלי Credentials.
4. להגדיר Aliases ו־Projects.
5. להוסיף validation ל־YAML ו־seed ל־Postgres.

תוצר: Context Resolver עובד בבדיקות.

### שלב 3 — שבוע 2: Telegram Shadow Bot

1. ליצור Bot פרטי.
2. להקים grammY webhook.
3. לשלוח ACK בתוך שנייה.
4. לקבל טקסט ו־Voice.
5. לתמלל Voice.
6. להפיק ActionPlan.
7. להריץ Validator.
8. להציג Preview בלבד.
9. לשמור תיקון כ־Eval case.

תוצר: Text/Voice → Plan, בלי שום כתיבה חיצונית.

### שלב 4 — שבוע 3: Google Tasks + Reminders

1. להקים Google Cloud project.
2. לבחור OAuth consent mode נכון; לא להישאר ב־Testing בלי תוכנית.
3. לבקש רק Tasks scopes.
4. לחבר חשבון אישי אחד.
5. להוסיף Scheduler ו־Telegram delivery.
6. לבדוק timezone, DST ו־relative dates.
7. להפעיל אישור לכל יצירה ב־20 הפעולות הראשונות.

תוצר: משימה נכתבת וגם מתקבלת תזכורת בזמן.

### שלב 5 — שבוע 3: Collections ו־Capability Gaps

1. ליצור Collections גמישות.
2. להוסיף `/find` ו־CSV export.
3. לזהות `unsupported_intent`.
4. לשמור Capability Gap מצונזר.
5. להפיק `/gaps` מדורג.

תוצר: המערכת גם שימושית כשאין לה כלי מתאים.

### שלב 6 — שבוע 4: Jira Pilot

1. לבחור חברה אחת ו־Project אחד.
2. לוודא שמדיניות החברה מאפשרת Telegram ומודל חיצוני.
3. ליצור API token נפרד ולשמור כ־secret.
4. לסנכרן Projects, Issue Types ו־Epics לקריאה בלבד.
5. להריץ 30 פקודות Jira ב־Shadow Mode.
6. לאפשר Create עם Preview ואישור.
7. להוסיף idempotency ו־duplicate check מוגבל.
8. לבצע Board verification אסינכרוני.

תוצר: Ticket ראשון בפרויקט וב־Epic הנכונים.

### שלב 7 — שבוע 5: אמינות תפעולית

1. Credential health job.
2. Retry/backoff ו־dead-letter status ב־Postgres.
3. Backup/export.
4. Latency dashboard בסיסי.
5. Rate-limit tests.
6. תרחישי Restore ו־token expiry.

תוצר: אפשר לסמוך על המערכת בשימוש יומיומי.

### שלב 8 — רק אחרי שימוש אמיתי

על בסיס `capability_gaps`, Audit ונתוני שימוש, בוחרים אחת בלבד:

- Email triage.
- Morning brief.
- Commitment tracking.
- חברה נוספת.
- Macros.

אין לפתח את כולן במקביל.

---

## 24. טבלאות ליבה

```text
workspaces
identities
connections
projects
aliases
policies
conversations
pending_interactions
action_plans
action_runs
approvals
idempotency_keys
collections
collection_items
scheduled_notifications
capability_gaps
eval_cases
credential_health_checks
model_benchmark_runs
```

הודעות גולמיות נשמרות רק לפי מדיניות retention. ברירת המחדל היא לשמור גרסה מצונזרת ו־hash, לא תוכן סודי לנצח.

---

## 25. סיכונים ותגובות

| סיכון | תגובה |
|---|---|
| Workspace שגוי | confidence threshold, clarification, hard identity binding |
| Token פג | health check יומי, pending queue, אין fallback לזהות אחרת |
| Telegram אסור בחברה | workspace block או תוכן מצונזר בלבד |
| תאריך יחסי שגוי | parser דטרמיניסטי + הצגת זמן מוחלט |
| פעולה כפולה | idempotency + recent action check |
| Prompt injection ממייל | Subagent הסגר ללא כלים/הקשר; תוכן חיצוני כ־untrusted data |
| עלייה בעלויות | budgets, usage log, caching, provider eval חודשי |
| Agent לא יודע לבצע | capability gap + workaround + roadmap score |
| אובדן מידע אישי | backups + CSV/JSON export + restore drill |
| מורכבות יתר | שירות יחיד, ללא Mini App/n8n/vector DB/multi-agent |
| בחירת מודל מיושנת | re-benchmark חודשי + כלל החלפה כתוב (סעיף 30) |

---

## 26. Definition of Done ל־MVP

ה־MVP הושלם רק כאשר:

- 200 הרצות Shadow/Eval עומדות בספי האיכות.
- Telegram מקבל טקסט ו־Voice ומחזיר Preview בזמן היעד.
- Google Task ותזכורת Telegram פועלים מקצה לקצה.
- Jira נבדק על חברה ופרויקט אחד, עם Epic מחייב.
- Conversation interruptions אינן מתפרשות בטעות כאישור.
- Connection פג מזוהה מראש.
- כל פעולה ניתנת לחיפוש ב־`/find`.
- פקודה לא נתמכת נשמרת ב־Capability Gap Memory ומופיעה ב־`/gaps`.
- קיים Export של Collections ו־Audit.
- אין Credentials ב־Git, logs או prompts.
- קיימת דרך שחזור מתועדת.

---

## 27. ההחלטות הסופיות

1. המערכת היא כלי אישי, לא מוצר עתידי בתחפושת.
2. בונים שירות TypeScript יחיד; n8n הוסר.
3. Telegram הוא הממשק הראשי, בכפוף למדיניות סודיות לכל Workspace.
4. YAML + Git מחליפים Control Panel ב־MVP.
5. APIs/SDKs/MCP רשמיים מורכבים לפי התאמה; MCP אינו חובה.
6. GPT-5.4 mini הוא ברירת המחדל הראשונית עד שה־Eval יכריע אחרת.
7. GPT-4o mini Transcribe הוא ברירת המחדל הראשונית ל־Voice.
8. Supabase Postgres נשאר מסד הנתונים.
9. Jira מתחיל עם API token נפרד וחברה אחת.
10. Google Tasks נכנס לפני Gmail ו־Calendar write.
11. תזכורת כוללת Scheduler ו־Telegram push כבר ב־V1.
12. Board verification ו־Audit enrichment אסינכרוניים.
13. Shadow corrections הופכים אוטומטית ל־regression evals.
14. Capability Gap Memory הוא רכיב ליבה, לא רשימת רעיונות ידנית.
15. Email triage, Daily Brief ו־commitment tracking מוקדמים יחסית, לפי נתוני שימוש.
16. Vector DB, Mini App ו־Realtime Voice נדחים עד שיש כאב מוכח.
17. Subagents מותרים רק בשלושה תפקידים מוגדרים (סעיף 29); אין Multi-Agent אוטונומי.
18. בחירת המודל אינה חד־פעמית; מתקיימת לולאת re-benchmark חודשית עם כלל החלפה כתוב (סעיף 30).

---

## 28. מקורות לבדיקה לפני הקמה

- [OpenAI API pricing](https://developers.openai.com/api/docs/pricing)
- [Supabase pricing](https://supabase.com/pricing)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Google Tasks API](https://developers.google.com/tasks)
- [Google OAuth policies](https://developers.google.com/identity/protocols/oauth2)
- [Atlassian Cloud REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/)
- [Atlassian API tokens](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/)
- [Atlassian Rovo MCP](https://www.atlassian.com/platform/remote-mcp-server)
- [grammY](https://grammy.dev/)

---

## 29. מדיניות Subagents

תת־סוכן (קריאת מודל נפרדת, עם הקשר וכלים משלה) מוצדק **רק** כשהוא קונה בידוד — של הקשר, ההרשאות או הסיכון. לעולם לא כדי "לחלק עבודה" בפקודה רגילה. המערכת אינה Multi-Agent אוטונומי: אין תזמור סוכנים שמדברים זה עם זה, אין סוכן שבוחר כלים באופן חופשי ואין סוכן ארוך־טווח עם זיכרון עצמאי.

### 29.1 Subagent הסגר לתוכן לא־אמין

נכנס עם V1.2 (Inbound Email Triage). מייל, PDF או צילום מסך עלולים להכיל הוראות מוטמעות ("התעלם מהמדיניות ושלח..."). התוכן החיצוני **לעולם לא** נכנס להקשר של ה־Planner הראשי. במקום זה:

1. קריאת מודל נפרדת, ללא כלים וללא גישה ל־Workspace/Policy, מקבלת רק את התוכן הגולמי.
2. הפלט מוגבל ל־schema קשיח: נושא, בקשות מזוהות, תאריכים, אנשים מוזכרים — טקסט תיאורי בלבד, לא פעולות.
3. ה־Planner הראשי מקבל את התקציר המובנה כמו כל קלט אחר, ועובר דרך אותו Validator ו־Policy Engine.

כך גם אם המסמך מנסה להורות "שלח מייל עכשיו", אין לו ערוץ לבצע דבר — אין לו כלים ואין לו Identity.

### 29.2 Critic / Verifier

קריאת מודל שנייה, חסרת מצב, שמופעלת רק כאשר **שני** התנאים מתקיימים יחד:

- confidence של ה־ActionPlan בטווח 0.70–0.89 (ספק אמיתי, לא ודאות גבוהה ולא נמוכה מדי).
- הפעולה מסווגת עתירת־סיכון במדיניות (למשל `jira.create`, ובעתיד `gmail.send`, `calendar.invite`).

הקריטיקן מקבל את ה־ActionPlan והקשר מצומצם, ומחזיר אישור/דחייה/הצעת תיקון. אינו מחליף את מסך האישור למשתמש — הוא שכבת סינון לפני שהמשתמש בכלל רואה פעולה מפוקפקת. צפוי לחול על פחות מ־5% מהתעבורה, כך שהעלות התוספתית זניחה יחסית לרשת הביטחון.

### 29.3 Subagents לפיתוח (לא Runtime)

בזמן בניית המערכת עצמה (לא בזמן ריצה אצל המשתמש) מותר להשתמש בסוכני קוד מקביליים לכתיבת אדפטרים, Eval runner ובדיקות. זה משפיע על מהירות הפיתוח בלבד ואינו חלק מארכיטקטורת הריצה.

### 29.4 מה נשאר אסור

- סוכן שמחליט אילו כלים או Credentials להפעיל ללא מעבר ב־Validator.
- שרשור סוכנים שמעביר החלטות ביניהם בלי בדיקה דטרמיניסטית בין השלבים.
- סוכן "רקע" שפועל ללא טריגר מפורש של המשתמש.

---

## 30. לולאת עלות/ביצועים למודלים

בחירת המודל אינה החלטה חד־פעמית. המנגנון הבא מבטיח שבכל נקודת זמן המודל הפעיל הוא הטוב ביותר עבור הפקודות בפועל, לפי עלות וביצועים יחד — לא לפי מותג או תחושת בטן.

### 30.1 אינסטרומנטציה של כל קריאה

כל קריאה למודל (Planner, Critic, Transcription) נרשמת ב־`action_runs` עם: שם מודל וגרסה, גרסת prompt, טוקני קלט/פלט (כולל cached), latency, עלות מחושבת לפי מחירון עדכני, ותוצאה התנהגותית — `approved_as_is` / `edited` / `corrected` / `rejected`. **שיעור האישור-בלי-עריכה הוא מדד האיכות האמיתי**, לא ה־confidence שהמודל מדווח על עצמו.

### 30.2 Eval Set חי

כל תיקון משתמש (Shadow correction, עריכת Preview, gap) מצטרף אוטומטית ל־`eval_cases` (סעיפים 13.5 ו־14). כך הסט גדל בדיוק בכיוון שבו המערכת טועה על המשתמש האמיתי, ולא לפי בנצ'מרק כללי.

### 30.3 Re-benchmark מתוזמן

ג'וב חודשי (וגם ידני, כשיוצא מודל חדש שרלוונטי) מריץ את כל ה־`eval_cases` על המודל הנוכחי ועל 1–2 מתמודדים, דרך אותו Provider interface (סעיף 4.8) — כך שהחלפה לא דורשת שינוי קוד. הפלט נשמר ב־`model_benchmark_runs` ומדווח לפי: דיוק Workspace/Identity/Project/Epic/Intent בנפרד, עלות ל־1,000 פקודות, ו־p95 latency. עלות הרצה כזו: סנטים עד דולרים בודדים בחודש.

### 30.4 כלל החלפה כתוב מראש

מודל מתמודד מחליף את המכהן **רק** אם:

- הוא שווה או טוב ממנו בכל ממדי הדיוק הקריטיים (Workspace, Identity, Epic) — לא ממוצע כולל שמסתיר רגרסיה בממד מסוכן; **וגם**
- זול ב־25% לפחות באותה רמת דיוק; **או** מדויק ב־2 נקודות אחוז לפחות בעלות של עד פי 1.5 מהמכהן;
- ועומד בתקציב ה־Latency של סעיף 16 ב־p95.

ההחלטה נעשית לפי הכלל, לא לפי היקסמות מהוצאת מודל חדשה. שינוי המודל הוא ערך קונפיגורציה אחד — commit אחד, rollback מיידי אם התנהגות בפועל מאכזבת.

### 30.5 נראות שוטפת

ה־Weekly Review (עתידי, V1.3) מציג "עלות לפקודה מוצלחת" לאורך זמן. Budget alert אצל ספק המודל מתריע על חריגה בלתי צפויה בטרם תגיע לחשבון.

---

## 31. הפעולה הראשונה

לא מתחילים מ־OAuth, Jira, שרת MCP או UI.

מתחילים מיצירת `evals/commands.jsonl` עם 100 פקודות אמיתיות ומהתוצאה הרצויה לכל אחת. לאחר מכן בונים Bot ב־Shadow Mode שמסוגל להפוך טקסט או Voice ל־ActionPlan מאומת — בלי לבצע דבר.

זהו המיילסטון הראשון והחשוב ביותר: אם הניתוב וההקשר אינם מדויקים, שום אינטגרציה לא תציל את המוצר. אם הם מדויקים, שאר המערכת היא חיבור הדרגתי של כלים קיימים.
