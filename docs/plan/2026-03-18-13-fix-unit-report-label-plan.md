## Fix Weird Unit Names In AI Reports

### Summary
Replace runtime `GetUnitName(...)` usage in commander/AI status reports with a stable unit report label derived from unit type and owner.

### Implementation
- Add `GetUnitReportLabel` in `common.eai`.
- Use it in `GetCurrentCommandReport`.
- Use it in `GetCurrentAICommandReport`.
- Keep existing null/dead handling unchanged.
- Update `CHANGELOG.md`.

### Verification
- Run `MakeTFT.bat`.
- Confirm the generated `common.ai` uses the new helper.
