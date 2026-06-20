# Why Prediction Markets — the Signal Thesis

The foundational reason this project exists. Polymarket alerts are used as an
early-warning signal for real-world events — not to monitor the markets for
their own sake.

## Key Facts

- **Polymarket alerts are used as a real-world early-warning signal, not for tracking the markets themselves** — because traders with money at stake (skin in the game) price in new information quickly, so a sharp move in a market's implied probability often means something is happening in the world before it reaches mainstream news. The alert is a prompt to go look, not a market-watching dashboard.
- **The *move* (a sudden jump), not just the absolute level, is the signal of interest** — because a rapid change is what suggests new information has entered the market. Example: a jump in *"Strait of Hormuz traffic returns to normal by July 31"* may reflect de-escalation signals ahead of the headlines. This is why edge-triggered threshold-crossing alerts (not level checks) are the right primitive.
- **Signals are treated as indications, not facts** — because markets can be thin (low liquidity), manipulated, or produce false signals. Every alert is a prompt to investigate, never something to act on blindly.

## Details

This thesis shapes the product. It explains why threshold-*crossing*
(edge-triggered) alerts are the right primitive rather than polling absolute
levels, why thin/low-liquidity markets are a known caveat rather than a bug to
fix, and why the tool optimizes for a timely nudge ("go look now") over
precision or completeness.

Connects to [[alerting-strategy-design]] (edge-triggered `once`/`hysteresis`
firing) and [[polymarket-integration-scope]] (binary-event scope; the Yes-token
probability is the signal being watched).

## Sources

- Clarified by the project owner, 2026-06-15: the reason for building the tool
  is prediction markets as a real-world signal (skin-in-the-game rationale),
  with explicit thin-market / manipulation / false-signal caveats. Surfaced
  while reframing the HTML documentation's overview.
