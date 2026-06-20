# Shared Supabase Project Namespacing

This tool does not own a Supabase project — it shares one with other
mini-projects, isolated by a dedicated Postgres schema. This file captures why
that shape was chosen and which Supabase namespaces must be prefixed to coexist.
The pattern is reusable: every future mini-project added to the shared project
should follow it.

## Key Facts

- **Multiple mini-projects share one Supabase project, isolated by a Postgres
  schema (`polymarket_alerts`), not by a separate project or database** —
  because the free tier allows only **2 active projects**, so dedicating one
  per mini-project doesn't scale. A *separate database* within the project was
  rejected because Supabase's tooling (PostgREST, supabase-js, Edge Functions,
  the dashboard) only ever connects to the single `postgres` database — a second
  database would be unreachable. A schema is the only workable isolation
  boundary, and `drop schema ... cascade` cleanly removes a whole mini-project.
- **Every project-global Supabase namespace is prefixed per mini-project** —
  because several namespaces are shared across *all* mini-projects in the one
  project and would otherwise collide: Edge Function names, Edge Function
  secrets, Vault secret names, and pg_cron job names. This project uses
  `polymarket-alerts-poll` (function), `POLYMARKET_ALERTS_TELEGRAM_BOT_TOKEN`
  (secret), `pma_project_url` / `pma_service_role_key` (Vault), and
  `pma-poll-every-5-min` (cron job).
- **The function-secret collision is the sharpest edge** — because Edge Function
  secrets are shared across *all* functions in the project (not scoped per
  function). A sibling mini-project setting a bare `TELEGRAM_BOT_TOKEN` would
  silently overwrite this tool's token for every function. Hence the
  project-prefixed secret name.
- **A custom schema must be added to the API's "Exposed schemas" list** —
  because PostgREST only serves exposed schemas, and the Edge Function reaches
  Postgres through PostgREST via supabase-js even when using the `service_role`
  key. The schema also needs `usage` + table privileges granted to
  `service_role`. Without exposure, the deployed function's queries fail with a
  schema-not-exposed error.

## Details

The supabase-js client is pinned to the schema once
(`createClient(url, key, { db: { schema: "polymarket_alerts" } })`), so query
code stays unqualified (`.from("alerts")`) while resolving inside the schema.

Vault secret names are global, and `vault.create_secret` errors on a duplicate
name — so the scheduling SQL is a one-time run, and prefixed names (`pma_*`)
keep it from clashing with a sibling project's secrets.

This relates to [[platform-choice]] (why Supabase at all) — that decision
assumed a free-tier project; this file records that the project is *shared*, not
dedicated.

## Sources

- Polymarket → Telegram alerts namespacing decision, 2026-06-15
  (see `docs/superpowers/specs/2026-06-14-polymarket-telegram-alerts-design.md`,
  section "Shared Supabase project & namespacing")
