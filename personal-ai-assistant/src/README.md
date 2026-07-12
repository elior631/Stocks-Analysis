# Src — שירות ה-TypeScript

הקוד מתחיל להיכתב ב**שלב 3** של המפרט (Telegram Shadow Bot), אחרי שה-Eval Dataset (שלב 1) והבסיס ב-Postgres (שלב 2) קיימים. אין לכתוב קוד לפני שיש baseline מדיד — ראו §31 במפרט: "אם הניתוב וההקשר אינם מדויקים, שום אינטגרציה לא תציל את המוצר".

## מבנה מתוכנן (§7)

```text
src/
├── bot/            Telegram webhook, grammY, ACK מיידי
├── conversation/    State machine (idle / awaiting_clarification / awaiting_approval / ...)
├── context/         Context Retrieval + Resolver (workspace/identity/project/alias)
├── planner/         קריאת LLM, Structured Output, Provider interface (§4.8)
├── validation/       Validator דטרמיניסטי (§10)
├── policy/           Policy Engine + Approval matrix
├── capabilities/      Capability Gap Memory (§13)
├── tools/
│   ├── jira/
│   ├── google-tasks/
│   └── collections/
├── reminders/        Scheduler + Telegram push (§12)
├── health/           Credential health checks (§18)
├── audit/            action_runs, /find (§19)
└── evals/            Eval runner + model benchmark (§30)
```

אין ליצור abstraction/interface כללי מעבר למה שנדרש בפועל (כלל מ-§7): כל הפשטה צריכה לפחות שני consumers אמיתיים או סיבה בטיחותית מפורשת.
