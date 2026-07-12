# Supabase — מסד הנתונים

תיקייה זו תתמלא ב־**שלב 2** של המפרט (`docs/personalaiassistantspec.md` §23) עם:

- `migrations/` — סכימת הטבלאות מ־§24 (workspaces, identities, connections, projects, aliases, policies, conversations, pending_interactions, action_plans, action_runs, approvals, idempotency_keys, collections, collection_items, scheduled_notifications, capability_gaps, eval_cases, credential_health_checks, model_benchmark_runs).
- `seed.ts` — טעינת הקבצים מ־`../config/*.yaml` לטבלאות המתאימות.

עדיין לא נפתח פרויקט Supabase בפועל. אין להריץ migration כלשהו לפני שה־Eval Dataset (`../evals/commands.jsonl`) מוכן ונבדק, לפי סדר העדיפויות בסעיף 31 של המפרט.
