# Speed Up AI Battle and Creep Pace

## Summary

Make AMAI chain actions faster by cutting idle transition time in the shared attack and creep flow, while preserving retreat safety and a small item-pickup window.

## Implementation Changes

- Add shared pacing settings in `common.eai` for regroup delay, target polling, point-goal polling, loot linger, post-attack handoff, and post-expansion creep delay.
- Reduce generic army regroup waits before normal attacks, commander attacks, queue attacks, desperation attacks, and alliance-target attacks.
- Keep Ancient uproot timing unchanged for Ancient of War creep rushes and ancient expansion attacks.
- Tighten `CommonSleepUntilTargetDeadAM` and `SleepUntilAtGoalAM` polling so targets and point attacks transition faster.
- Shorten `SleepInCombatAM` lingering after enemies are gone, but keep a brief item-pickup window.
- Reduce the extra wait after expansion-creep clearing and after a finished player-target attack.

## Test Plan

- Run `MakeTFT.bat`, `MakeROC.bat`, and `MakeREFORGED.bat`.
- Verify in-game that creep-to-creep and creep-to-attack transitions happen faster.
- Verify point attacks hand off more quickly after reaching the destination.
- Verify AI still picks up nearby items during the shorter linger window.
- Verify retreat and overextension behavior remain unchanged.

## Assumptions

- Use a moderate pacing profile.
- Keep a short loot window instead of removing loot collection waits entirely.
- Apply the change through shared logic only; no race-specific pacing changes are needed.
