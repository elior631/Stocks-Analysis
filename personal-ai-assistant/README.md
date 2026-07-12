# Personal AI Command Layer

עוזר AI אישי רב־מערכתי (Telegram → Jira / Google Tasks / Collections).

- מפרט מלא: [`docs/personalaiassistantspec.md`](docs/personalaiassistantspec.md) (גרסה 2.1)
- סטטוס נוכחי: **שלב 0 הושלם** (שלד תיקיות), **שלב 1 בתהליך** (איסוף 100 פקודות אמיתיות ל־Eval — ראו `evals/README.md`).

## מבנה התיקייה

```text
personal-ai-assistant/
├── docs/     מפרט המוצר, שמתעדכן לאורך הפרויקט
├── config/   Workspaces / Identities / Projects / Policies כ־YAML (ללא Credentials)
├── evals/    Dataset של פקודות אמיתיות + schema להערכת מודלים
├── src/      קוד השירות (מתחיל בשלב 3 של המפרט)
└── supabase/ Migrations למסד הנתונים (מתחיל בשלב 2)
```

## הצעד הבא

לפי סעיף 31 במפרט: למלא את `evals/commands.jsonl` (100 פקודות אמיתיות שלך, בעברית) לפני כתיבת כל שורת קוד. ראו `evals/README.md` להנחיות מדויקות ולטבלת הקטגוריות הנדרשת.

אין בתיקייה הזו Credentials, API tokens או Secrets בשום שלב — הכלל חל גם על קבצי הדוגמה.
