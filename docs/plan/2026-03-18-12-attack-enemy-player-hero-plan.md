## Change attack-hero commander command to target enemy player, not hero index

- Replace `ATTACK HERO <index>` and `FORCE ATTACK HERO <index>` with player-targeted hero commands.
- Dialog/player argument should choose an enemy player first, matching existing commander enemy-player selection flow.
- Resolve that player to their first living, visible, non-illusion hero.
- If that player has no living hero, show a clear ally message and keep the current command unchanged.
- Update command table, help text, English strings, changelog, and rebuild with `MakeTFT.bat`.
