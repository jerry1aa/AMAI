# Show AI Command State Plan (2026-03-18)

## Goal
Extend the existing `SHOW COMMAND` commander command so it also reports the AI's own currently chosen autonomous state, instead of only reporting player-issued commander orders.

## Problem
The current implementation reports only commander-layer flags such as:
- `attack_player`
- `attack_point`
- `attack_unit`
- `queue_running`

That is useful for player-to-AI orders, but it does not answer what the AI itself is currently doing. The AI's autonomous behavior is spread across multiple systems, so the solution needs to summarize stable high-level state instead of exposing one short-lived internal variable.

## Decisions
- Keep the existing commander report; do not remove or replace it.
- Add a second AI-state report section to `SHOW COMMAND`.
- Prefer stable high-level state over noisy short-lived micro details.
- Report AI state by priority, so the output remains deterministic and easy to read.

## AI State Priority
1. `Retreating`
   - when `isfleeing` or equivalent retreat-control state is active.
2. `Defending threatened town`
   - when `town_threatened` is active and the AI is not currently overriding it with a forced commander attack.
3. `Attacking`
   - when `attack_running` is active.
4. `Idle / regrouping`
   - fallback when no stronger state applies.

## Optional Context To Append
- Current local focus target from `focus_fire_unit` when valid and alive.
- Current strategy name from `GetCurrentStrategyName()`.
- Commander override marker when `commanded_attack_active` is true.

## Implementation
1. Update `common.eai`:
   - keep `GetCurrentCommandReport` for commander state,
   - add a new helper for AI autonomous state, for example `GetCurrentAICommandReport`,
   - combine both in the `SHOW COMMAND` output.
2. Pull AI state from existing globals instead of adding a new persistent command-tracking system:
   - `attack_running`
   - `town_threatened`
   - `isfleeing`
   - `focus_fire_unit`
   - strategy getters already used by the strategy-report commands.
3. Format output as concise multi-line ally chat text, for example:
   - `Commander command: Attack point (x,y)`
   - `AI command: Defending threatened town`
   - `AI focus target: Archmage`
   - `AI strategy: Fast Bears`
4. Update any user-facing text/help only if needed.
5. Update `CHANGELOG.md`.

## Validation
- Run `MakeTFT.bat`.
- Confirm `SHOW COMMAND` still works with:
  - no commander order active,
  - commander order active,
  - autonomous defend state,
  - retreat state,
  - normal attack-running state.
- Check that missing or dead focus targets fall back cleanly without invalid text.

## Notes
- `focus_fire_unit` should be treated as extra context, not the primary AI command, because it changes rapidly during battle.
- The report should stay readable in chat; avoid dumping too many internal flags at once.
