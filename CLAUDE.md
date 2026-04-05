# CLAUDE.md

## High level summary

- All files ending with `.eai` are AMAI AI source code in its JASS language.
- AI code is compiled to the `Scripts` folder (look there to see generated output used by maps).
- `Common.j` and `Natives.j` are standard Warcraft 3 AI/runtime functions.
- `Blizzard.j` contains standard map functions; `Blizzard.eai` is the AMAI "commander" logic for maps.
- Many build helpers are provided as `.bat` and `.pl` scripts in the project root (e.g. `MakeREFORGED.bat`, `MakeROC.bat`, `MakeTFT.bat`).

## Important rules & constraints

- **JASS language rule**: local variables must always be declared at the start of a function. Any edits to JASS-style `.eai` code must respect this rule.
- **Function order matters**: functions cannot be called unless they are imported or defined earlier in the file.
- **Never edit `Scripts/` directly** — change source `.eai` files and re-run the project build to regenerate compiled output.
- `Common.j`, `Natives.j`, and `Blizzard.j` are built-in Warcraft 3 code — do not modify them; they exist for reference to hardcoded functions.
- **Update `CHANGELOG.md`** when changes are made.
- **`git push` workflow**: first run `git pull --no-rebase` (merge), then push.
- **Plans**: when executing a plan, save it under `docs/plan/` with a date + daily index in the filename (default `YYYY-MM-DD-01`, then `YYYY-MM-DD-02`, etc.).
- **Version coverage**: if a change relates to game-version-specific behavior or data, cover all supported versions (`TFT`, `ROC`, and `REFORGED`) unless the user explicitly scopes the change to fewer versions.
- **Agent workflow**: the main agent owns research, planning, repo-rule compliance, review, and final integration; one subagent may be used for bounded coding/testing work when delegation is useful, but subagent use is optional for small or tightly coupled tasks.
- **No casual judgements**: do not make general claims about game mechanics, race behaviour, or code behaviour without first verifying against actual source files or data (e.g. `StandardUnits.txt`, `.eai` source, compiled scripts). Check before asserting.

## Useful file locations

- AI source code: `*.eai` files (mostly under the top-level `AMAI/` folder).
- Compiled/packaged AI used by maps: `Scripts/` folder (after running the build scripts).
- Build helpers and packaging scripts: top-level `.bat` files (e.g. `MakeREFORGED.bat`) and Perl scripts like `InstallToDir.pl`.
- Electron app and UI: `Electron/` (contains Angular + Electron tooling).

## Quick workflow for making an AI change

1. Find the source `.eai` file(s) to change. Use filename search for the feature you need to edit.
2. Make minimal, well-scoped changes. Keep local variable declarations at the head of functions.
3. Run the relevant build script to compile AI code into `Scripts/` (see examples below).
4. Verify compiled output in `Scripts/` and run a map/test to exercise the change.
5. Add or update tests or small reproducible map scenarios when possible.

## Build commands (run from repository root)

```powershell
# Compile for Reforged
.\MakeREFORGED.bat

# Compile for TFT
.\MakeTFT.bat

# Compile for ROC
.\MakeROC.bat

# Build the Electron main (TypeScript compile)
# npm run electron:build
```
