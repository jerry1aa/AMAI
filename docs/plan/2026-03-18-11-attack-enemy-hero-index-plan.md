## Add commander enemy-hero index attack command

- Add `ATTACK HERO <index>` and `FORCE ATTACK HERO <index>` commander commands.
- Interpret the index as 1-based over living enemy heroes only, skipping dead heroes.
- Resolve the requested hero into `target_unit` and reuse the existing attack-unit execution path.
- Show a clear ally message when no living enemy hero exists for the requested index.
- Update `Commands.txt`, `Languages/CommanderHelp.txt`, `Languages/English/CommandsTrans.txt`, and `CHANGELOG.md`.
- Rebuild with `MakeTFT.bat`.
