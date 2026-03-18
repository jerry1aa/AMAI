## Undead Rich-Economy Upgrade Boost

### Goal
Speed up Undead graveyard attack and armor research when the AI has surplus resources.

### Scope
Apply the same behavior to all supported game versions:
- TFT
- ROC
- REFORGED

### Changes
- In each `Undead/BuildSequence.ai`, add a global rich-economy boost block for:
  - `uUPG_UNHOLY_STR`
  - `uUPG_UNHOLY_ARMOR`
  - `uUPG_CR_ATTACK`
  - `uUPG_CR_ARMOR`
- Trigger the boost only when both gold and lumber are comfortably high.
- Keep the change limited to priority boosts.

### Verification
- Run `MakeTFT.bat`
- Run `MakeROC.bat`
- Run `MakeREFORGED.bat`
- Confirm no new parse errors
