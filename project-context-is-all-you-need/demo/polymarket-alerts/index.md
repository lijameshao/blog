# Project Context Index

Quick-reference map of where project knowledge lives. Each entry links to a context file.

Polymarket → Telegram threshold alert tool. Phase 1 is a personal script
(Supabase cron + Edge Function); a multi-user web app is the planned phase 2.

## Purpose
- [Why prediction markets — the signal thesis](why-prediction-markets.md) - the foundational why: prediction markets as a real-world early-warning signal

## Business Logic
- [Alerting strategy design](alerting-strategy-design.md) - once vs hysteresis, flip-flop deadband, edge-triggered firing

## Architecture
- [Data model decoupling](data-model-decoupling.md) - why markets and alerts are separate tables, and where decoupling stops
- [Platform choice](platform-choice.md) - why Supabase over AWS Lambda and Vercel
- [Shared Supabase project namespacing](shared-supabase-namespacing.md) - schema isolation + prefixing for sharing one project across mini-projects

## Integrations
- [Polymarket integration scope](polymarket-integration-scope.md) - binary-only scope, Gamma API price quirks

## Constraints
- [Alert delivery reliability](alert-delivery-reliability.md) - send-first/persist-on-200 ordering invariant
