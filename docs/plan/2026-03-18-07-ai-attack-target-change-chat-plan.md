# AI Attack Target Change Chat Plan (2026-03-18)

## Goal
Notify allied players when the AI changes its high-level attack target, without spamming chat for short-lived local focus-fire retargets.

## Problem
The AI changes local battle targets very frequently during focus-fire control. If chat is sent on every such change, allied players will be flooded with messages. The useful signal is the high-level attack target change: player, point, unit, or captain destination.

## Decisions
- Do not report `focus_fire_unit` changes.
- Report only high-level attack-target changes.
- Reuse the information already exposed by `GetCurrentAICommandReport`.
- Add throttling so repeated updates do not spam ally chat.

## Reporting Scope
Report only when the effective high-level AI attack target changes:
- commander/player-target attack
- commander/point-target attack
- commander/unit-target attack
- internal AI attack destination via `lastcaptainx`, `lastcaptainy`

Do not report:
- local focus-fire retargets
- tiny movement updates while continuing the same attack

## Implementation
1. Add cached report state in `common.eai`, for example:
   - last reported AI target text
   - last report time
2. Add a helper that returns a stable high-level attack-target label:
   - player name
   - point coordinates
   - unit name / dead / none
   - fallback captain target position
3. Add a notifier helper, for example:
   - compare current target label with cached label
   - require a minimum interval such as 15 to 30 seconds between ally-chat reports
   - send `DisplayToAllies("AI target changed: ...")` only when label meaningfully changes
4. Hook the notifier into stable high-level attack entry points:
   - `AttackMoveKillAAM`
   - `AttackMoveKillXYAAM`
   - queue transitions if needed
5. Update `CHANGELOG.md`.

## Validation
- Run `MakeTFT.bat`.
- In-game verify:
  - AI attacking a different player triggers one ally message
  - AI switching to a different point triggers one ally message
  - AI changing to a different unit target triggers one ally message
  - normal focus-fire retargets do not spam chat
  - repeated reissue of the same attack target does not spam chat

## Notes
- Target equality for positions should be coarse enough to avoid spam from tiny coordinate drift.
- The notifier should be independent of `SHOW COMMAND`; `SHOW COMMAND` remains a pull-based status report, while this feature is push-based.
