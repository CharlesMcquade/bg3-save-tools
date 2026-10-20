# BG3 Journal Reader

**bg3-save-tools**: a tested procedure for reading a *Baldur's Gate 3* save's real campaign state: active party, every quest's current objective and **`QuestVisitedSteps`**, the ordered trail of quest steps you actually triggered. Use it to answer "what did I choose last playthrough?" or "has this already happened in my run?" without loading the game.

> Unofficial fan project. Not affiliated with or endorsed by Larian Studios. Uses Norbyte's open-source [lslib / Divine](https://github.com/Norbyte/lslib). Tested with Divine v1.20.4 on PC (Steam) saves from game version 4.1.1, and may change with future patches. Contains no game code or assets.

This repo has no script. Divine already does the unpacking well, so the value here is the exact invocations, the save structure and the traps between them.

## Why this exists

*Baldur's Gate 3* remembers far more than its journal shows. Hundreds of hours in, or starting a second run, it's hard to answer simple questions: did I side with them last time? Did that scene already fire? Why is this quest still open? Each quest's `QuestVisitedSteps` trail in the save answers them, step by step.

- **Where am I?** Every quest's current objective and which ones the game considers finished.
- **What did I do last time?** Read the final save of an earlier run and compare its quest trails with your current run's choice points.
- **Has this already happened?** Check whether a step is in the trail before you reload or go looking for it.
- **What's next?** Give the journal JSON and `SKILL.md` to an AI agent. The save supplies your exact position; the agent maps it to [bg3.wiki](https://bg3.wiki) and suggests the next step at the spoiler level you choose. This repo contains no walkthrough.

## What a save tells you

| Source | Contents |
|---|---|
| `SaveInfo.json` | Party members (origin, class, subclass, level, XP), current area, difficulty, game version |
| `Globals.lsj` → `Journal` | Per-quest current objective, `QuestVisitedSteps` trail and completed/active category buckets |
| `Globals.lsj` → `DialogLogs` | Conversation history |

A visited step proves you *saw* that beat, not that you *chose* it. Some steps fire automatically. Check the quest on [bg3.wiki](https://bg3.wiki) before reading intent into a trail.

## Quickstart (Windows)

1. Download **ExportTool** from [lslib releases](https://github.com/Norbyte/lslib/releases/latest). `Divine.exe` is at `Packed/Tools/Divine.exe` inside the zip.
2. Find the save: `%LOCALAPPDATA%\Larian Studios\Baldur's Gate 3\PlayerProfiles\Public\Savegames\Story\<save folder>\<name>.lsv`
3. Unpack it, then convert the globals:

```bat
Divine.exe -g bg3 -a extract-package  -s "C:/.../Story/<folder>/<name>.lsv" -d "C:/work/out"
Divine.exe -g bg3 -a convert-resource -i lsf -o lsj -s "C:/work/out/Globals.lsf" -d "C:/work/out/Globals.lsj"
```

4. Read `save.regions.Journal` from `Globals.lsj` with any JSON tool. Every value is wrapped as `{"type": ..., "value": ...}`, so unwrap `value` before matching.

`SKILL.md` covers the JSON paths, the gotchas (apostrophe folder, folder ≠ file name, absolute paths) and how to read a trail. It's also usable as an AI-agent skill.

## License

MIT
