# MisterWeapon modpack: modernization plan

## Structure

Six Space Engineers 1 mods. Each is its own git repo, and each has a **dev** and a **final** Workshop version.

```
missil-misterplace (brain: guided-missile logic, scripts only)
 ├── missilSLBM-misterplace   (blocks: SLBM launcher, AMissile/SLBMMissile/WaterMissile)
 ├── IronDomeClassic          (original Iron Dome turret + launcher, data only; the brain's scripts drive them)
 ├── LauncherX                (blocks + its own script for now)
 └── IronDomeX                (blocks + its own script for now)
AntennaIronDome              (standalone antenna + radar script; no brain dependency)
```

On disk the three Iron Dome repos (`IronDomeClassic`, `IronDomeX`, `AntennaIronDome`) sit together in the
`IronDome/` folder of the workspace; the brain, SLBM and LauncherX are at the workspace root. Repo-relative doc links
to the brain therefore use `../../missil-misterplace`.

| Mod | Repo? | Dev Workshop ID | Final Workshop ID |
|---|---|---|---|
| missil-misterplace | ✅ `missil-misterplace/` | 3514275550 | 3496380795 |
| missilSLBM-misterplace | ✅ `missilSLBM-misterplace/` | none yet (first dev Publish creates it) | 3685206054 |
| IronDomeClassic | ✅ `IronDome/IronDomeClassic/` | none yet | none yet (not published) |
| LauncherX | ✅ `LauncherX/` | none yet | none yet (not published) |
| IronDomeX | ✅ `IronDome/IronDomeX/` | none yet | none yet (not published) |
| AntennaIronDome | ✅ `IronDome/AntennaIronDome/` | none yet | none yet (not published) |

The brain's old pre-git folder (`missil-misterplace-dev/`) was identical to the repo's `Mod/` and was moved to
`%APPDATA%\SpaceEngineers\ModBackups\missil-misterplace-dev-source-20261117-224010`. Edit only in
`missil-misterplace/Mod/`; the game's dev mod links there.

## Phases

1. **One git repo per mod** ✅: `Mod/` holds what ships, Git LFS holds binary assets, and the current state
   (published, where applicable) is the first commit (`release/baseline`). The brain has its unreleased dev
   changes as a second commit.
2. **Dev/final deploy** ✅: `deploy.json` + `deploy.ps1` in every repo. Dev is a junction (live edits); final is a
   snapshot copy from a clean commit plus a `release/*` tag. See
   [missil-misterplace/docs/deployment.md](../missil-misterplace/docs/deployment.md).
3. **Claude-only workflow** (IDE dropped; the user works through Claude Code only): no `.csproj` or IDE project. API
   correctness is checked with the `se-dev-mod` skill (Mod API whitelist, code search) and `se-dev-game-code`.
   `se-dev-mod` is set up ✅ (prepared, Graphify off). The in-game compile (world load) is the real
   check, and errors are in `%APPDATA%\SpaceEngineers\SpaceEngineers_<date>_<time>.log` (newest file).
4. **Block mods depend on the brain** (in progress: LauncherX requires it and its missile is aquatic/SLBM ammo;
   SLBM is not declared yet, IronDomeX has the `requires` entry; see the dependency section in
   [deployment.md](../missil-misterplace/docs/deployment.md)): move LauncherX and IronDomeX logic into the brain, and replace
   hard-coded ammo subtypes with registration (by subtype config or by mod messages). Block mods then
   carry only data.

   Done so far (2026-11-17): the Iron Dome blocks, ammo, blueprints and icons moved out of the brain into
   `IronDomeClassic` (data only). Their scripts (`MissileTurret.cs`, `MissileLauncher.cs`, `IDMissile` guidance) stay in
   the brain, so the split is a data split; moving the logic too is part of the registration work above. The brain now
   ships scripts only. Also: `IronDomeX` is registered in the brain's Iron Dome logic (block in the `MissileLauncher`
   descriptor, ammo in `GuidedMissileSystem.InterceptorAmmoSubtypes`) and now requires the brain.
5. **Share on GitHub** ✅ (2026-11-18, public repos under `Artur-Mi`): see the section below.

See [status.md](status.md) for what is done, tested and still open.

## GitHub sharing

Goal: a friend clones one repo and gets the docs plus all four mods at versions known to work together.
Decision: a base repo in this folder, with the four mods as **submodules** (not ignored folders). Reason: ignoring the
mod folders would leave a clone without the mods; a plain `git add .` of nested repos records broken gitlinks.

Done 2026-11-18 (steps 1 to 3 and 6; repos are **public** by the user's choice; `gh` CLI installed and logged in as `Artur-Mi`). Submodule URLs are relative (`../<mod>.git`), so they follow whatever protocol the base repo was cloned with. `deploy.json` holds no machine paths (only the Steam owner ID and Workshop IDs), so no example file was needed; `deploy.ps1` was made OS-agnostic instead.

Steps, in order:
1. Create a GitHub repo per mod (private unless told otherwise) and add it as `origin`; push `main` and the
   `release/*` tags, plus Git LFS objects (`git lfs push --all origin`). Check LFS quota first.
2. Check each `deploy.json` for machine-specific paths; if any, commit a `deploy.example.json` and ignore the local one.
3. In this folder: `git init`, add a `.gitignore` for machine-specific files, then
   `git submodule add <mod-url> <folder>` for each mod (the existing folders are picked up; `.gitmodules` is written).
   Commit the workspace files (`CLAUDE.md`, `docs/`) and push as `MisterWeaponModpack`.
4. Private repos: add the friend as collaborator on all five.
5. Friend clones with `git clone --recurse-submodules <url>` and needs Git LFS installed.
6. Update the docs: the workspace `CLAUDE.md` currently says this folder is not a git repo, and links into mod repos
   (for example `missil-misterplace/docs/deployment.md`) resolve only with submodules checked out.

Day to day: commit and push inside a mod as usual. To publish that version to the friend, `git add <mod>` and commit in
the base repo (moves the pinned pointer); the friend runs `git pull --recurse-submodules`.
Fallback if submodules are too heavy: a base repo with only the docs and a `setup.ps1` that clones the four mods.

## Open questions
- `AntennaIronDome` is now a repo in the workspace (standalone, no brain dependency). Should `M61IronDomeX` (in the game's Mods folder, not in this workspace) join the pack too?
