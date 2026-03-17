# AI Attack Report Detail Plan (2026-03-18)

## Goal
Improve `GetCurrentAICommandReport` so `AI command: Attacking` includes meaningful target detail instead of focus-fire information.

## Decisions
- Remove focus-fire reporting from the AI command report.
- When the AI is attacking, report stable attack context:
  - commander-selected player / point / unit when active,
  - otherwise the active captain attack destination from `lastcaptainx` and `lastcaptainy`.
- Keep strategy context in the report.

## Implementation
1. Update `common.eai`:
   - replace `focus_status` with a more general attack-detail field,
   - add attack detail only when `attack_running` is true,
   - prefer commander target info when available,
   - fall back to captain destination coordinates.
2. Leave retreat/defend/idle reporting unchanged.
3. Update `CHANGELOG.md`.

## Validation
- Run `MakeTFT.bat`.
- Confirm `SHOW COMMAND` reports:
  - `AI command: Attacking | target player (...)`
  - `AI command: Attacking | target point (...)`
  - `AI command: Attacking | target unit (...)`
  - or `AI command: Attacking | target position (...)` when using internal AI attack flow.
