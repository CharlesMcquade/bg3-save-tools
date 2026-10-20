---
name: bg3-save-tools
description: Read Baldur's Gate 3 (Larian .lsv) save state with Norbyte's Divine - party, journal objectives, and QuestVisitedSteps route history. Use when a player asks what quest stage they are at, what they chose on a prior run, or whether an event already happened.
---

# Baldur's Gate 3 — reading saves with Divine

Save data is ground truth for **what** happened in a run. It can't tell you **why** a player chose something.

## Tooling

- **Divine.exe** from Norbyte's lslib **ExportTool** — https://github.com/Norbyte/lslib/releases/latest (tested v1.20.4). The binary is at `Packed/Tools/Divine.exe` inside the zip, NOT the zip root.
- Windows tool. Run it where the saves live, or copy the `.lsv` to a Windows box.

## Locating saves

- `%LOCALAPPDATA%\Larian Studios\Baldur's Gate 3\PlayerProfiles\Public\Savegames\Story\`, with one folder per save. Newest by mtime = latest quicksave/autosave.
- ⚠ **Apostrophe-dir gotcha:** the profile dir is `Baldur's Gate 3` (WITH the apostrophe). Some installs also have `Baldurs Gate 3` (no apostrophe) for launcher/data. List `Larian Studios\` and check which one contains `PlayerProfiles\...\Savegames` first. The wrong guess looks like "no saves".
- Each save is a **folder whose name is not the inner filename**: folder `<Name>-<id>__QuickSave_12` contains `QuickSave_12.lsv` (+ `QuickSave_12.WebP` thumbnail). The inner name is the part after the last `__`, so list the folder first.
- `.lsv` is an **LSPK package** (v18 for BG3), not a zip. unzip/7z see nothing in it.

## Procedure

1. **Extract the package**
   ```
   Divine.exe -g bg3 -a extract-package -s "C:/.../Story/<FOLDER>/<INNER>.lsv" -d "C:/work/out"
   ```
   - Pass the inner `.lsv` **file**. The folder path gives "Access denied".
   - `-g bg3` is required for `extract-package`.
   - Use Windows **absolute** paths (`C:/...`). From git-bash, `/c/...` fails with "Cannot proceed without absolute path"; convert with `cygpath -w`.
2. **Convert the binary resource to JSON**
   ```
   Divine.exe -g bg3 -a convert-resource -i lsf -o lsj -s "C:/work/out/Globals.lsf" -d "C:/work/out/Globals.lsj"
   ```
   - Use `-o lsj` and a `.lsj` destination. Divine rejects both `-o json` and a `.json` extension.
3. **Parse** `Globals.lsj` with any JSON reader. It's tens of MB for a late-game save, so load it once and slice.
   - Over Windows OpenSSH, `scp` wants Windows-style remote paths (`host:C:/work/out/Globals.lsj`), while plain `ssh` commands under git-bash use `/c/...`.

## What's readable

- **`SaveInfo.json`** (plain JSON in the extracted package): active party (origin, class, subclass, level, XP, position), current area, difficulty, game version, save name.
- **`Globals.lsj`** → `save.regions.Journal`:
  - `QuestCategories[0].QuestCategory[]`: bucket membership per quest (`CompletedQuests`, area-named active buckets).
  - `Quests[0].Quests[0].QuestsProgress[]`: per-quest `MapKey` → `MapValue[0].Quest[]` with `ObjectiveID` (current objective) and **`QuestVisitedSteps`**, the chronological list of steps the player triggered (who gave a lead, whether a pact was taken, whether a location was destroyed).
  - `DialogLogs` region: conversation history. There are also many global flag families for deeper digs.
- Also in the package: `meta.lsf` (tiny, metadata only, **no journal**), `StorySave.bin` (opaque), `LevelCache/`.

## Interpreting QuestVisitedSteps

- Step names are **internal IDs** (e.g. `<QUESTPREFIX>_<Location>_<Event>`), not journal prose. Map them to the quest's wiki page before drawing conclusions.
- **A visited step proves the player saw it, not that they chose it.** Auto-triggers fire without input. Confirm choice-vs-trigger with the wiki.
- To answer "what did I do last run?", read the prior run's final save and compare its trails to the current run's choice points.

## Caveats

- **Every scalar in `.lsj` is type-wrapped**: `{"type": ..., "value": ...}`. Unwrap `node["value"]` before matching. Matching the wrapper silently returns zero hits.
- Category buckets update on **area load**, so a pull reflects the last area transition, not the current minute.
- Treat as strong evidence; still verify mechanics claims against bg3.wiki.
