## Night Elf Rich-Economy Upgrade Boost

### Goal
Speed up Night Elf Hunter's Hall weapon and armor research when the AI has surplus resources.

### Scope
Apply the same behavior to all supported game versions:
- TFT
- ROC
- REFORGED

### Changes
- In each `Elf/BuildSequence.ai`, add a global rich-economy boost block for:
  - `uUPG_STR_MOON`
  - `uUPG_MOON_ARMOR`
  - `uUPG_STR_WILD`
  - `uUPG_HIDES`
- Trigger the boost only when both gold and lumber are comfortably high.
- Keep the change limited to priority boosts.

### Verification
- Run `MakeTFT.bat`
- Run `MakeROC.bat`
- Run `MakeREFORGED.bat`
- Confirm no new parse errors
