# Show Current Command Plan (2026-03-18)

## Goal
Add a new Commander command (`SHOW COMMAND`) that reports the AI's current command state in chat.

## Decisions
- Syntax: `SHOW COMMAND`.
- Scope: current attack/queue state plus restriction flags.
- Output: one chat line to allies with target details when available.

## Implementation
1. Add command table entry in `Commands.txt` with a new command id in misc range.
2. Add `GetCurrentCommandReport` in `common.eai` to summarize:
   - active attack mode (player/point/unit),
   - queue mode and current queued target,
   - force marker and restrictions (`no_attack`, `no_creep_attack`, `no_player_attack`).
3. Hook command id in `cmd_misc` to call `DisplayToAllies(GetCurrentCommandReport())`.
4. Add help text in `Languages/CommanderHelp.txt`.
5. Add English translation row in `Languages/English/CommandsTrans.txt`.
6. Add changelog entry in `CHANGELOG.md`.

## Validation
- Run `MakeTFT.bat`.
- Verify parse success and no new errors.

