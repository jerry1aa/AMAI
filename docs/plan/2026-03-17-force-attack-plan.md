# Plan: Distinguish Player-Commanded vs AI-Chosen Attacks + Force Attack Command

## Summary
- Add explicit flags for player-commanded attacks (and a force variant).
- Ensure player-commanded attacks are tracked and cleared when the command finishes or is canceled.
- Add a new "Force Attack" command/button in the Commander dialog (ESC / -cmd UI).

## Key Changes
- Command state flags
  - Add globals in common.eai for:
    - commanded_attack_active (true while a player-issued attack is in effect)
    - commanded_attack_force (true only for force commands)
  - Set/clear in command handlers:
    - cmd_attack and cmd_queue set commanded_attack_active for player commands.
    - Force commands set commanded_attack_force = true.
    - Clear both flags in cmd_cancel (attack/all), cmd_attack stop, queue cancel, and after commanded attack finishes in SingleMeleeAttackAM.
- Force behavior (Never retreat)
  - Skip SleepUntilTownDefended(...) if commanded_attack_force is true.
  - Prevent retreat control from starting during forced attacks by gating EnableRetreatControl in DoAttackJobs.
  - Keep normal retreat/defense behavior for non-force commands and AI-chosen attacks.
- Commander dialog button (root menu)
  - Add new command rows in Commands.txt for:
    - Force Attack Player
    - Force Attack Here (point)
    - Force Attack Selected Unit
  - Use a distinct command ID range (e.g., 40–42) to avoid collisions.
  - Dialog text uses "Force Attack" as the top-level label with hotkey F, and sub-options "Enemy / Current screen / Selected unit".
  - Add handling for new command IDs in cmd_attack in common.eai.
- Documentation
  - Update Languages/CommanderHelp.txt to document the new force attack command.
- Changelog
  - Add an entry to CHANGELOG.md.

## Tests / Validation
1. Normal attack command: commanded_attack_active set and cleared; AI still defends normally.
2. Force attack command: AI does not return to town from threat/retreat and completes the command.
3. Cancel/Stop/Queue cancel: commanded flags reset, AI returns to normal behavior.
4. Commander dialog: new Force Attack root button appears with hotkey F.

## Assumptions
- The command dialog UI is the in-game Commander dialog (ESC / -cmd).
- Force attack applies to player, point, and selected unit commands (no forced queue commands).
- Force attacks bypass both town defense and retreat control ("never retreat").
