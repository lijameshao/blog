# Project Context Changelog

## 2026-06-14
- Created [why-prediction-markets.md] - the signal thesis: prediction markets as a real-world early-warning signal (the foundational why); added Purpose section to index
- Created [shared-supabase-namespacing.md] - schema isolation + prefixing for sharing one Supabase project across mini-projects (2-project free-tier cap)
- Updated [platform-choice.md] - noted the Supabase project is shared, not dedicated; cross-linked namespacing
- Created [alerting-strategy-design.md] - once vs hysteresis strategies, flip-flop deadband rationale, deferred strategies
- Created [data-model-decoupling.md] - markets/alerts split, channel-not-split boundary, expire_at over is_active
- Created [platform-choice.md] - Supabase chosen over AWS Lambda and Vercel
- Created [alert-delivery-reliability.md] - send-first/persist-on-200 invariant
- Created [polymarket-integration-scope.md] - binary-only scope, Gamma API quirks
