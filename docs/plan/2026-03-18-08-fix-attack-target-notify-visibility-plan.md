## Fix missing AI attack-target change notifications

- Keep the existing high-level attack-target hooks unchanged.
- Remove the dependency on the global `chatting` toggle inside `ReportAIAttackTargetChange` so these status messages still appear even when normal AI chatter is disabled.
- Keep duplicate suppression and cooldown behavior unchanged.
- Rebuild with `MakeTFT.bat` to verify no parse errors.
