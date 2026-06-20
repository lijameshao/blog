# Polymarket Integration Scope

Constraints and assumptions about how the tool reads prices from Polymarket,
and the scope boundary that follows from them.

## Key Facts

- **Phase 1 supports binary Yes/No events only — one slug maps to one Yes
  probability** — because a Polymarket *event* (the URL the user pastes) can
  contain multiple *markets* (outcomes), and handling multi-outcome events means
  deciding which outcome's price to watch. That complexity is out of scope until
  there's a concrete need; binary events cover the driving use case (e.g.
  "Strait of Hormuz traffic returns to normal by July 31").
- **Price comes from the Gamma API, which returns `outcomePrices` as a
  *stringified* JSON array, not a real array** — so the value must be parsed
  before use, and a parse failure is treated as a fetch failure (skip the market
  this tick) rather than a crash. Worth recording because it's a non-obvious
  gotcha that looks like clean JSON until it isn't.
- **The Yes-token price IS the implied probability** — a price of 0.82 means the
  market implies an 82% chance, which is what the threshold compares against
  directly (no conversion needed).

## Sources

- Polymarket → Telegram alerts design session, 2026-06-14
  (see `docs/superpowers/specs/2026-06-14-polymarket-telegram-alerts-design.md`)
