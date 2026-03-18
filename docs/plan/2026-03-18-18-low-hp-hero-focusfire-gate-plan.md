## Low-HP Hero Focus-Fire Gate

### Goal
Prevent low-HP heroes from joining focus-fire attack orders.

### Changes
- Update `Jobs/FOCUSFIRE_CONTROL.eai`
- Add a direct HP threshold inside `HeroCanFocusFireTarget`
- Use a conservative threshold above the normal flee threshold so heroes stop joining focus-fire before emergency save logic triggers

### Verification
- Run `MakeTFT.bat`
- Run `MakeROC.bat`
- Run `MakeREFORGED.bat`
- Confirm no new parse errors
