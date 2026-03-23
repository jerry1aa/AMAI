# Hero Town Portal Battle-Aware Rework

## Summary
- Block self-save TP for level-1 heroes.
- Keep defensive TP-to-town available regardless of that level gate.
- Stop self-save TP from forcing the whole army to retreat.
- Prevent low-HP retreat handling from being overwritten by threatened-town TP in the same hero micro tick.
- Only commit the army to a TP-driven retreat after the defensive TP cast has actually started.

## Key Changes
- Add `hero_self_tp_min_level = 2` in shared config.
- Apply the level gate only to `SaveHero` / `ACTION_TP`.
- Split self-save TP from defensive town TP:
  - self-save TP keeps the hero alive but does not collapse the attack state
  - defensive town TP may still pivot the main army home
- Move army-retreat commitment for defensive TP into the teleport job so a failed/interrupted TP does not prematurely end the attack.
- Return immediately after `SaveHero(...)` in the low-HP branch so the same cycle cannot issue a conflicting threatened-town TP.

## Test Plan
- Build `MakeTFT.bat`, `MakeROC.bat`, and `MakeREFORGED.bat`.
- Verify a level-1 hero does not self-TP from `SaveHero`.
- Verify a level-2+ hero can still self-TP.
- Verify self-save TP by the main hero does not auto-retreat a winning army.
- Verify defensive TP to a threatened town still works and only pivots the army once the TP order is active.
- Verify low-HP retreat no longer gets overwritten by threatened-town TP in the same tick.

## Assumptions
- The level gate is only for self-save TP.
- Army retreat remains controlled by `RETREAT_CONTROL` except for the dedicated threatened-town TP path.
