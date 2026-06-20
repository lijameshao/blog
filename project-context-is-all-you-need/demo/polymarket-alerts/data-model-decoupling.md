# Data Model Decoupling

Why the schema splits the observed market from the alert rule, and where that
decoupling deliberately stops.

## Key Facts

- **Markets and alerts are separate tables even though phase 1 has a single
  event** — because one market can carry many alert rules (e.g. "ping me at
  80%", "ping me if it drops below 20%", and later other users' rules). Keeping
  them separate means each unique market is polled **once per tick** rather than
  once per rule, which is both cheaper on the Polymarket API and the correct
  multi-tenant shape for the planned web app. Done now, while the DB is empty,
  to avoid a painful migration later.
- **The notification channel (`telegram_chat_id`) is intentionally NOT factored
  into its own table** — because there is exactly one channel type (Telegram)
  and one recipient. A channels table is YAGNI until multiple channels or
  recipients exist; splitting it now would add a join and buy nothing. This
  marks the deliberate boundary of the decoupling.
- **A single nullable `expire_at` replaces a separate `is_active` flag** —
  because `is_active = false` and "past expiry" are two ways to express the same
  "don't poll this" state, which is confusing. One `expire_at` expresses both
  time-bound auto-expiry and manual kill (set it to `now()`). A reversible
  *pause* was judged a web-app-phase need and deferred to a future `status`
  enum, rather than reintroducing two overlapping on/off fields.

## Details

The model treats a **market** as an observed fact (its price is whatever it is)
and an **alert** as a subjective rule layered on top (threshold, strategy,
where to notify, current state). `alert_log` records each firing against the
alert that fired.

The market↔alert relationship is a plain one-to-many and obvious from the table
names, so no ER diagram is warranted — the value here is the *reasoning* for the
split, not the shape.

Phase-2 additions this model is shaped to absorb without restructuring:
`direction` (above/below) on alerts, a `status` enum for reversible pause, a
notification-channel table, and multi-user ownership.

## Sources

- Polymarket → Telegram alerts design session, 2026-06-14
  (see `docs/superpowers/specs/2026-06-14-polymarket-telegram-alerts-design.md`)
