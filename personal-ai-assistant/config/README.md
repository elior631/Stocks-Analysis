# Config — Workspaces / Identities / Projects / Policies

לפי §8 במפרט: קונפיגורציה מנוהלת כ־YAML ב־Git, לא ב־UI. **קבצים אלה לעולם אינם מכילים Secrets, API tokens או סיסמאות** — רק מזהים, כינויים ומדיניות. חיבור בפועל (OAuth / API token) מתבצע בשלב 4/6 דרך secret manager של פלטפורמת האירוח, לא כאן.

## קבצים

- `workspaces.example.yaml` — עולמות מבודדים (חברות, מיזמים, אישי).
- `identities.example.yaml` — כתובות אימייל/משתמשים לכל workspace.
- `projects.example.yaml` — הקשרים עסקיים/אישיים, כולל aliases ומיפוי Jira.
- `policies.example.yaml` — מטריצת אישורים לפי workspace ופעולה (§23 טבלת ברירת מחדל).

## איך להתחיל

1. העתק כל קובץ `*.example.yaml` ל־`*.yaml` (ה־`.gitignore` בהמשך יחריג את הגרסה האמיתית אם יש בה מידע רגיש שאינך רוצה ב־Git ציבורי — אבל ה־Repo הזה כבר פרטי, כך שברירת המחדל היא לשמור הכול, כולל השמות האמיתיים, תחת Git).
2. מלא את הערכים האמיתיים: שמות חברות, אימיילים, פרויקטים, כינויים.
3. השתמש באותם מזהים (`workspace_id`, `project_id`) בתוך `evals/commands.jsonl` כדי ששני הקבצים יתאימו.

שלב 2 במפרט (בסיס הנתונים) ייקח את אותם קבצים ויטען אותם ל־Postgres דרך seed script.
