# Show Active vs Chosen Strategy Plan (2026-03-18)

## Goal
Improve `GetCurrentAICommandReport` so the strategy text clearly distinguishes between:
- the currently active executed strategy (`strategy`)
- the AI's stored selected/base strategy (`chosen_strategy`)

## Problem
The current report uses `GetCurrentStrategyName()`, which reflects only `strategy`.
That can be misleading when:
- commander build overrides temporarily change `strategy`
- `chosen_strategy` remains different in the background

In that case, users cannot tell whether the shown strategy is:
- the active final strategy being executed now, or
- the AI's long-term chosen strategy.

## Decisions
- Treat `strategy` as the active/final executed strategy.
- Treat `chosen_strategy` as the AI's chosen/base strategy.
- If both are equal, report a single strategy name.
- If they differ, report both values and mark the active one as an override state.

## Output Format
- If equal:
  - `| strategy NormalHuman`
- If different:
  - `| strategy Rifle [active override] | chosen NormalHuman`

## Implementation
1. Update `common.eai` inside `GetCurrentAICommandReport`:
   - read both `strategy` and `chosen_strategy`
   - format strategy text based on equality/inequality
2. Keep the existing AI status and attack-detail reporting unchanged.
3. Update `CHANGELOG.md`.

## Validation
- Run `MakeTFT.bat`.
- Confirm `SHOW COMMAND` output in both cases:
  - `strategy == chosen_strategy`
  - `strategy != chosen_strategy` after a commander build override
- Verify no variable-name collisions are introduced in generated `common.ai`.
