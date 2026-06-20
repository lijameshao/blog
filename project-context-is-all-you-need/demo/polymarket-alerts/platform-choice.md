# Platform Choice

Why the tool runs entirely on Supabase, with Vercel reserved for a later phase.

## Key Facts

- **Supabase was chosen over AWS Lambda for compute + scheduling + state** —
  because a single platform handles all three (pg_cron for scheduling, Edge
  Functions for compute, Postgres for state), and its free tier (500K Edge
  Function invocations/month) far exceeds the ~8,600/month a 5-minute poll
  needs. The AWS equivalent (Lambda + EventBridge + DynamoDB) works and is also
  free-tier-viable, but spreads the system across three services for no benefit
  at this scale.
- **Vercel was rejected for the scheduling/polling job specifically** — because
  its Hobby (free) plan caps cron frequency at hourly, which is too slow for a
  useful threshold alert. Vercel is still the intended host for the **frontend**
  in the web-app phase; it just can't own the poll loop.
- **The job is a scheduled poll, not an always-on server** — because the
  workload is "check N markets every 5 minutes," which a cron-triggered
  serverless function serves at zero idle cost; a long-running process would
  burn a free VM doing nothing between ticks.
- **The Supabase project is shared with other mini-projects, not dedicated to
  this tool** — because the free tier caps active projects at 2. Isolation is by
  Postgres schema; see [[shared-supabase-namespacing]] for the full rationale
  and the namespaces that must be prefixed.

## Sources

- Polymarket → Telegram alerts design session, 2026-06-14
  (see `docs/superpowers/specs/2026-06-14-polymarket-telegram-alerts-design.md`)
- Shared-project namespacing decision added 2026-06-15
