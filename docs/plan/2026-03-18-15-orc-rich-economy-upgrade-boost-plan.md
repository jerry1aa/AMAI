## Orc Rich-Economy Upgrade Boost

### Goal
Speed up Orc forge weapon/armor research when the AI has surplus resources.

### Scope
Apply the same behavior to all supported game versions:
- TFT
- ROC
- REFORGED

### Changes
- In each `Orc/BuildSequence.ai`, add a global rich-economy boost block for:
  - `uUPG_ORC_MELEE`
  - `uUPG_ORC_RANGED`
  - `uUPG_ORC_ARMOR`
- Trigger the boost only when both gold and lumber are comfortably high.
- Keep the change limited to priority boosts; do not add extra Forge construction.

### Verification
- Run `MakeTFT.bat`
- Run `MakeROC.bat`
- Run `MakeREFORGED.bat`
- Confirm no new parse errors
