## Game-Start Economy Preset Dialog

### Summary
Add a host-only startup dialog that appears after the game-mode choice and lets the host select one economy preset for all AMAI players. The preset sets both AI gold and lumber income percentages together.

### Key Changes
- Extend `Blizzard.eai` startup flow with a second game-start dialog for economy presets.
- Add six fixed presets: `50`, `60`, `70`, `80`, `90`, `100`.
- Send the chosen raw percent to all AMAI players through a new AI misc command `72`.
- Handle command `72` in `common.eai` by setting both `economy_gold_income_percent` and `economy_lumber_income_percent`.
- Keep `economy_income_interval` unchanged and leave `100` as the effective default/no-modifier option.
- Add or reuse localized dialog labels through the existing language-init pattern.

### Test Plan
- Build `MakeTFT.bat`.
- Build `MakeROC.bat`.
- Build `MakeREFORGED.bat`.
- Verify the host sees the economy dialog after game-mode selection.
- Verify all AMAI players receive the selected preset and apply the same percent to gold and lumber income.

### Assumptions
- The dialog is host-only and applies one global preset to all AMAI players.
- The feature uses preset buttons, not a numeric input dialog.
- English fallback text is acceptable for any untranslated languages.
