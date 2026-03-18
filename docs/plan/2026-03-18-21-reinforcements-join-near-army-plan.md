# Prevent Far Reinforcements From Stalling the Main Army

## Summary

Keep the main army moving during attacks by separating distant reinforcements from the active assault group. New units should move toward the current army first, and only join the assault group once they are close enough.

## Implementation Changes

- Add shared reinforcement settings in `common.eai` for join radius and move radius.
- Add a distance-limited regroup helper that rebuilds the assault group using only nearby units around the current army anchor.
- Add a reinforcement-forwarding helper that orders far assault candidates to move toward the current main army / captain anchor.
- Replace the mid-attack periodic `FormGroupAM(...)` refreshes with the new nearby-only regroup plus reinforcement forwarding.
- Keep full regroup at attack start unchanged.

## Test Plan

- Run `MakeTFT.bat`, `MakeROC.bat`, and `MakeREFORGED.bat`.
- Verify that when the main army attacks and new units spawn at base, the main army keeps moving instead of waiting for the far units.
- Verify that far reinforcements start moving toward the army and join once they get near the frontline.
- Verify that retreat and player-controlled unit protections still work.

## Assumptions

- Use the tracked `main_army` location as the primary reinforcement anchor.
- Only mid-attack reform becomes distance-limited; initial attack-start regroup remains full.
