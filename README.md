# bg3-save-tools

A procedure for reading your **actual** *Baldur's Gate 3* campaign state (party, quest stages, and the exact route you took through every quest) straight from a save file, using Norbyte's open-source **lslib / Divine** tool.

> **Disclaimer:** Unofficial fan tooling based on reverse-engineering; not affiliated with Larian / Awaken Realms / Questline. May break on game updates. No game assets redistributed.

There is no script here on purpose. Divine does the heavy lifting; this repo documents the workflow and the save structure.

## What you can learn from a save

- **Active party**: origin, class, subclass, level, XP, location (`SaveInfo.json`)
- **Every quest's current objective** and its **`QuestVisitedSteps`**: the chronological trail of steps you actually triggered. Useful for "what did I choose last run?"
- Quest category buckets (completed vs area-active), dialog logs, global flags

## Quickstart

1. Download **ExportTool** from [Norbyte/lslib releases](https://github.com/Norbyte/lslib/releases/latest) (v1.20.4 at time of writing). `Divine.exe` is at `Packed/Tools/Divine.exe` inside the zip.
2. Follow [`SKILL.md`](SKILL.md): extract the `.lsv` → convert `Globals.lsf` → `.lsj` → read `save.regions.Journal`.

License: MIT
